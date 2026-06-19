.. _qualcomm_hoya:

#################
Hoya architecture
#################

The Hoya family targets Qualcomm application processors. It currently covers the
``kodiak`` and ``lemans`` chipsets.

On top of the :ref:`common platform support <qualcomm>`, Hoya enables an 8-core
Cortex-A (ARMv8) configuration with a GICv3 interrupt controller
(``CFG_ARM_GICV3``).

Drivers and services
*********************
The following drivers and services are available on the Hoya architecture:

	- **RAMBLUR inline memory protection (v3)**
	  (``CFG_QCOM_RAMBLUR_PIMEM_V3``) providing anti-rollback, integrity and
	  confidentiality protection for secure memory windows.
	- **Secure PRNG** hardware random number generator (``CFG_QCOM_PRNG``) used
	  as the entropy source for ``hw_get_random_bytes()``.
	- **Qualcomm clock driver** (``CFG_DRIVERS_QCOM_CLK``) built on the OP-TEE
	  clock framework (``CFG_DRIVERS_CLK``).
	- **Command DB** (``CFG_QCOM_CMD_DB``), a read-only shared database used to
	  look up RPMh resource addresses and metadata.
	- **RPMh client** (``CFG_QCOM_RPMH_CLIENT``) to vote for shared resources
	  (clocks, regulators, etc.) through the RPMh subsystem. It depends on the
	  Command DB driver.
	- **QFPROM fuse provisioning** (``CFG_QCOM_QFPROM`` /
	  ``CFG_QCOM_QFPROM_FUSEPROV``) for reading and blowing one-time-programmable
	  fuses. Fuse provisioning is enabled by default on secure builds and
	  depends on the Command DB and RPMh client drivers.

Chipsets
********

Kodiak
======
In addition to the Hoya features above, the ``kodiak`` chipset enables the
**Peripheral Authentication Service (PAS)** pseudo TA (``CFG_QCOM_PAS_PTA``).
The PAS PTA authenticates and brings up remote subsystems such as the audio
DSP (LPASS/QDSP6), the compute DSP (Turing) and the Wi-Fi processor subsystem
(WPSS). It also applies the MX voltage-rail workaround required for QFPROM fuse
blowing on this chipset (``CFG_QFPROM_MX_RAIL_WA``).

Lemans
======
The ``lemans`` chipset enables the Hoya drivers and services described above,
including the Qualcomm clock driver and QFPROM fuse provisioning.
