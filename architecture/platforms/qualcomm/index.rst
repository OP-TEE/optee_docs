.. _qualcomm:

########
Qualcomm
########

OP-TEE supports Qualcomm SoCs through the ``plat-qcom`` platform. The platform
is organised around *architecture families*, where each family groups a set of
chipsets (``PLATFORM_FLAVOR``) that share the same CPU topology, drivers and
memory layout.

See :ref:`qualcomm_security_disclaimer` for the security expectations that
apply across all Qualcomm platforms.

Supported architectures
***********************
The following architecture families and chipsets are currently supported:

	- :ref:`qualcomm_hoya` – application processors
		- ``kodiak``
		- ``lemans``
	- :ref:`qualcomm_bobcat` – networking processors
		- ``ipq96xx``
		- ``ipq52xx``

.. note::

	Driver and feature support for both architecture families is still a
	work in progress. The per-architecture pages reflect the current state
	of the upstream port and are expected to grow over time.

To build for a given chipset, select the Qualcomm platform and the matching
flavor, for example:

.. code-block:: bash

	$ make PLATFORM=qcom PLATFORM_FLAVOR=kodiak

Common platform features
*************************
The following features are enabled for every Qualcomm chipset, regardless of the
architecture family:

	- **Boot flow** based on Trusted Firmware-A (``CFG_WITH_ARM_TRUSTED_FW``)
	  with an AArch64 secure core (``CFG_ARM64_core``) using a 40-bit physical
	  address space (``CFG_CORE_LARGE_PHYS_ADDR``).
	- **GIC** interrupt controller (``CFG_GIC``).
	- **Secure time** sourced from the physical counter ``CNTPCT``
	  (``CFG_SECURE_TIME_SOURCE_CNTPCT``).
	- **GENI serial engine UART** console driver (``CFG_QCOM_GENI_UART``). The
	  UART is shared with the non-secure world, so the ready-wait period is
	  tunable per platform via ``CFG_QCOM_GENI_UART_RDY_WAIT_USEC``.
	- **Arm Crypto Extensions** acceleration (``CFG_CRYPTO_WITH_CE``), which in
	  turn enables CE-accelerated AES, AES-GCM and SHA-1/SHA-256.
	- The **Hardware Unique Key** length is set to 32 bytes
	  (``CFG_HW_UNIQUE_KEY_LENGTH``). Most chipsets do not yet provide their own
	  ``tee_otp_get_hw_unique_key()`` implementation, so the default (test) key
	  from the OP-TEE core is used unless a board-specific provider is added.

Platform-dependent features
****************************
Some features vary across Qualcomm chipsets rather than being available
everywhere. The most significant is the **Peripheral Authentication
Service (PAS)** (see :ref:`qualcomm_pas_authentication`), currently only
available on the Hoya family, where it differs in scope and authentication
strength between chipsets; see :ref:`qualcomm_hoya` for how Kodiak and
Lemans compare. Other chipset-specific features, such as QFPROM, XPU and
watchdog support, are covered on the relevant architecture-family page.

Per-architecture documentation
******************************
Refer to the architecture-specific pages below for the drivers, services and
configuration available on each architecture family and its chipsets:

.. toctree::
	:maxdepth: 1

	hoya
	bobcat

Security architecture topics
*****************************
Some security-relevant content is documented separately because it applies
across, or may eventually apply across, more than one architecture family:

.. toctree::
	:maxdepth: 1

	pas_authentication
	qfprom_provisioning
	security_disclaimer
