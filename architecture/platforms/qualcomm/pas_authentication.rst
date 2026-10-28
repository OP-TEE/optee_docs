.. _qualcomm_pas_authentication:

###################
PAS authentication
###################

Overview
********
Several Qualcomm chipsets include on-SoC coprocessors, such as audio,
compute, Wi-Fi and camera subsystem processors, that boot from firmware
images loaded and authenticated at runtime. The Peripheral Authentication
Service (PAS) is the OP-TEE component that authenticates those images and
releases the coprocessor from reset. It acts on behalf of the Linux kernel's
remoteproc framework, which runs in the Rich Execution Environment (REE).

Architecture
************
PAS consists of one TA and two PTAs:

	- The **Qualcomm PAS Trusted Application (TA)** is the only component
	  visible to the REE. It accepts only kernel clients, including
	  remoteproc, and calls the PAS core pseudo TA (PTA) directly. When
	  authentication support is enabled, it also calls the fuse PTA and verifies
	  certificates/signatures according to the secure-boot fuse state.
	- The **PAS core PTA** (``CFG_QCOM_PAS_PTA``) drives the
	  subsystem-specific clock and reset sequencing needed to bring a
	  coprocessor up. With authentication support enabled, it also checks
	  ELF-header and loaded-segment hashes against metadata supplied by the
	  PAS TA before starting the subsystem.
	- The **fuse PTA** (``CFG_QCOM_FUSE_PTA``) gives the PAS TA a restricted,
	  read-only view of secure-boot fuse state. It is only used where
	  authentication support is enabled.

Both pseudo TAs accept requests only from the PAS TA; neither is reachable
directly from the REE or from other TAs.

In this platform integration, the Linux kernel runs at EL2 and is responsible
for configuring the translation tables that give each coprocessor access to
its assigned memory regions. It must also protect the firmware memory during
authentication and startup through the platform's MMU/SMMU configuration.

Authentication and bring-up flow
*********************************
The simplified flow below omits session and memory setup. It shows successful
requests; an authentication failure prevents subsystem startup.

.. uml::

    participant "Linux remoteproc" as Linux
    participant "Qualcomm PAS TA" as TA
    participant "Fuse PTA" as Fuse
    participant "PAS core PTA" as Core

    Linux -> TA: Initialize firmware image
    TA -> Core: Check subsystem support
    Core --> TA: Supported
    opt Authentication support enabled
      TA -> Fuse: Read secure-boot state and policy
      Fuse --> TA: State and policy
      opt Secure boot not explicitly disabled
        TA -> Fuse: Read root of trust
        Fuse --> TA: Root certificate hash
        TA -> TA: Verify certificates and signature
      end
    end
    TA --> Linux: Image initialized

    Linux -> TA: Authenticate and start loaded firmware
    opt Authentication support enabled
      TA -> Core: Verify ELF-header and loaded-segment hashes
      Core --> TA: Verified
    end
    TA -> Core: Start subsystem
    Core --> TA: Started
    TA --> Linux: Success

On the platforms currently supported, firmware authentication and subsystem
bring-up both happen entirely within OP-TEE; no separate secure-monitor
component is involved.

Platform support
****************
PAS is currently available on the :ref:`Hoya <qualcomm_hoya>` chipsets:

	- ``kodiak`` brings up the audio, video, compute and Wi-Fi subsystem
	  processors; no firmware-image verification is performed.
	- ``lemans`` brings up the audio, video and camera subsystems, two
	  compute DSPs and two general-purpose DSPs. It enables certificate-based
	  signature authentication support (``CFG_QCOM_PAS_AUTH``), using the
	  fuse PTA to read the hardware root of trust and secure-boot policy.

With ``CFG_QCOM_PAS_AUTH`` enabled, certificate/signature verification is
skipped only when the secure-boot fuses explicitly report that secure boot
is disabled. Firmware segment hashes are still checked in that case. An
unreadable secure-boot state does not bypass signature verification.

Limitations
***********
When signature verification is required, encrypted or Qualcomm-countersigned
firmware images are rejected because those formats are unsupported. Detailed
hardware programming information, including the firmware image format, is
outside the scope of this platform overview. Public technical reference
manuals for these SoCs are expected to become available soon. Links to the
relevant manuals will be added here when they are published.

See :ref:`qualcomm_security_disclaimer` for the platform-wide security
expectations this authentication relies on.
