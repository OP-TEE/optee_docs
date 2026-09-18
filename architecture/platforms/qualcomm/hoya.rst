.. _qualcomm_hoya:

#################
Hoya architecture
#################

The Hoya family targets Qualcomm application processors. It currently covers the
``kodiak`` and ``lemans`` chipsets.

On top of the :ref:`common platform features <qualcomm>`, Hoya enables an
8-core Cortex-A (ARMv8) configuration with a GICv3 interrupt controller
(``CFG_ARM_GICV3``).

Drivers and services
*********************
The following drivers and services are available on the Hoya architecture:

	- **RAMBLUR inline memory protection (v3)**
	  (``CFG_QCOM_RAMBLUR_PIMEM_V3``) providing anti-rollback, integrity and
	  confidentiality protection for secure memory windows.
	- **Secure RNG** hardware random number generator (``CFG_QCOM_CSRNG``) used
	  as the entropy source for ``hw_get_random_bytes()``.
	- **Qualcomm clock driver** (``CFG_DRIVERS_QCOM_CLK``) built on the OP-TEE
	  clock framework (``CFG_DRIVERS_CLK``).
	- **QFPROM** (``CFG_QCOM_QFPROM``) for reading one-time-programmable
	  fuses, with boot provisioning controlled by
	  ``CFG_QCOM_QFPROM_FUSEPROV``. Enablement differs per chipset; see below
	  and :ref:`qualcomm_qfprom_provisioning`.
	- **Peripheral Authentication Service (PAS)**
	  (``CFG_QCOM_PAS_PTA``) for authenticating and bringing up remote
	  subsystem firmware. See :ref:`qualcomm_pas_authentication` for the
	  overall architecture; chipset-specific subsystem coverage is described
	  below.

QFPROM supplies fuse-backed authentication data and supports provisioning at
boot. Both Kodiak and Lemans use Command DB to identify shared power resources
and RPMh to request programming supplies. Kodiak additionally needs an MX-rail
vote coordinated with the Always-On Processor (AOP). See
:ref:`qualcomm_qfprom_provisioning` for the service architecture, image
preparation, and provisioning outcomes.

Chipsets
********
Kodiak and Lemans differ on two security-relevant points:

.. csv-table::
	:header: "Chipset", "Signature authentication support", "Hardware Unique Key"

	"Kodiak", "Not enabled", "Default test key"
	"Lemans", "Supported", "Hardware (HWKM)"

Kodiak
======
PAS on ``kodiak`` brings up remote subsystems such as the audio DSP
(LPASS/QDSP6), the compute DSP (Turing), the video codec (IRIS) and the
Wi-Fi processor subsystem (WPSS), without certificate-based signature
authentication.

Boot fuse provisioning defaults to enabled on secure
(non-``CFG_INSECURE``) builds and enables QFPROM. Kodiak has no PAS fuse-read
service enabled by default.

Lemans
======
PAS on ``lemans`` brings up the audio DSP (LPASS/QDSP6), two compute DSPs
(Turing), two general-purpose DSPs (GPDSP), the video codec (IRIS) and camera
subsystems.

This chipset enables certificate-based signature authentication support
(``CFG_QCOM_PAS_AUTH``), backed by a restricted fuse-read service
(``CFG_QCOM_FUSE_PTA``). With that support enabled, certificate/signature
verification is skipped only when the secure-boot fuses explicitly report
that secure boot is disabled; firmware segment hashes are still checked.
See :ref:`qualcomm_pas_authentication` for details.

This chipset also derives its Hardware Unique Key from the Hardware Key
Manager (``CFG_QCOM_HWKM``) instead of using the OP-TEE core's default key.

QFPROM is enabled for either boot provisioning or the fuse-read service used
by PAS authentication. Boot provisioning defaults to enabled on secure builds;
the PAS fuse-read path can also enable QFPROM on insecure builds.
