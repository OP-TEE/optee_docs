.. _qualcomm_bobcat:

###################
Bobcat architecture
###################

The Bobcat family targets Qualcomm networking processors. It currently covers
the ``ipq96xx`` and ``ipq52xx`` chipsets.

Drivers and services
*********************
On top of the :ref:`common platform features <qualcomm>`, the following
drivers and services are available on the Bobcat architecture:

	- **Secure watchdog** (``CFG_QCOM_SEC_WDOG``) that resets the device if
	  the secure world stops responding.
	- **XPU-based bus protection** (``CFG_QCOM_XPUV4``) that restricts
	  non-secure access to secure DRAM and the diagnostic log buffer.

Chipsets
********

ipq96xx
=======
5-core configuration with a GICv3 interrupt controller (``CFG_ARM_GICV3``).

ipq52xx
=======
4-core configuration. This chipset also exposes the platform random-number
generator through the generic hardware RNG pseudo TA (``CFG_HWRNG_PTA``);
``ipq96xx`` does not currently enable this service.
