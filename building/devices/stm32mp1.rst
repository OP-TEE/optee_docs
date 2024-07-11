.. _stm32mp1:

########
STM32MP1
########

The instructions here will tell how to run OP-TEE on one of the supported
STM32MP1 boards.

Supported boards
****************

+---------------------+--------------------+------------+-------------------------------+
| Board Name          | Manufacturer       | Boot media | Hardware Description          |
+=====================+====================+============+===============================+
| `STM32MP135F-DK`_   | STMicroelectronics | SDcard     | `Wiki STM32MP135x-DK`_        |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157A-DK1`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157x-DKx`_       |
+---------------------+                    |            |                               |
| `STM32MP157D-DK1`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-DK2`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157x-DKx`_       |
+---------------------+                    |            |                               |
| `STM32MP157F-DK2`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-EV1`_  | STMicroelectronics | SDCard (1) | `Wiki STM32MP157x-EV1`_       |
+---------------------+                    |            |                               |
| `STM32MP157F-EV1`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+

(1): STM32MP157x-EV1 boards also integrate an eMMC device, a NOR flash and a
Nand flash the system can boot on. OP-TEE distribution however only supports
booting from the SDcard slot.

Build instructions
******************

Follow the instructions at ":ref:`get_and_build_the_solution`".

Configuration switch ``PLATFORM`` can be used to specify the target device
as listed in table below:

+------------------------+--------------------------------------+
| Board Name             | Build configuration directive        |
+========================+======================================+
| `STM32MP135F-DK`_      | ``PLATFORM=stm32mp1-135F_DK``        |
+------------------------+--------------------------------------+
| `STM32MP157A-DK1`_     | ``PLATFORM=stm32mp1-157A_DK1_SCMI``  |
| `STM32MP157D-DK1`_     | or ``PLATFORM=stm32mp1-157A_DK1``    |
+------------------------+--------------------------------------+
| `STM32MP157C-DK2`_     | ``PLATFORM=stm32mp1-157C_DK2_SCMI``  |
| `STM32MP157F-DK2`_     | or ``PLATFORM=stm32mp1-157C_DK2``    |
+------------------------+--------------------------------------+
| `STM32MP157C-EV1`_     | ``PLATFORM=stm32mp1-157C_EV1_SCMI``  |
| `STM32MP157F-EV1`_     | or ``PLATFORM=stm32mp1-157C_EV1``    |
+------------------------+--------------------------------------+

When the build completes, generated image file sdcard.img can be found
in the generated binary images directory ``../out/bin/`` from build
root path. The images is a GPT multipartition image you can raw copy
to the target SDcard using a tool like dd.

A usual short fecth/build/load shell sequence is like the one below:

.. code-block:: bash

  $ repo init -u https://github.com/OP-TEE/manifest.git -m stm32mp1.xml
  $ repo sync
  $ cd build
  $ make toolchains
  $ make PLATFORM=stm32mp1-157C_DK2_SCMI all
  $ dd if=../out/bin/sdcard.img of=/dev/sdX conv=fdatasync status=progress
  $ sgdisk -e /dev/sdX

Command ``sgdisk -e`` fixes the GPT backup data which location depends on
storage device effective size.

Alternate configurations
************************

The build makefile for STM32MP1 platforms (stm32mp1.mk) proposes some
extra configuration switch for specific purpose.

STM32MP15 SCMI and non-SCMI variants
====================================

In order to enable security on STM32MP15 platforms, some device resources
must be assigned to OP-TEE. In this configuration, non-secure world (e.g.
U-Boot bootloader and Linux kernel) use SCMI services to access the secured
resource (e.g. clocks and reset controllers). Therefore STM32MP15 boards
come each with 2 variant configurations:

* ``PLATFORM=stm32mp1-157A_DK1``, ``PLATFORM=stm32mp1-157C_DK2``
  and ``PLATFORM=stm32mp1-157C_EV1`` do not enable chip root secure
  hardening and U-Boot/Linux DTS files do not rely on OP-TEE SCMI services.

* ``PLATFORM=stm32mp1-157A_DK1_SCMI``, ``PLATFORM=stm32mp1-157C_DK2_SCMI``
  and ``PLATFORM=stm32mp1-157C_EV1_SCMI`` enable chip root secure hardening
  and U-Boot/Linux DTS files enable use of OP-TEE SCMI services.

STM32MP15 and OP-TEE pager
==========================

On STM32MP15 products, OP-TEE core default executes in a secure 256kB internal
RAM (named SYSRAM). The size of the internal RAM may be impacting regarding
OP-TEE pager performances. STM32MP15 product lines embed other internal RAMs
that are initially dedicated to the chip Cortex-M co-processor firmware but
can be assigned to OP-TEE to enlarge OP-TEE pager page pool and enhance
OP-TEE performances.

Build configuration switch ``WITH_SRAM1_PAGER_POOL=y|n``, when enabled (``y``),
assigns SRAM1 to OP-TEE, adding 128kB of internal secure RAM.
``WITH_SRAM1_PAGER_POOL`` is default enabled (but not mandated) for platform
flavors ``PLATFORM=stm32mp1-157C_*_SCMI`` and default disabled for other
flavors.

Another way to assign one or more SRAMx (x=1..4) is to set OP-TEE OS
configuration directive CFG_TZSRAM_SIZE to cover the SRAMs assigned to OP-TEE:

- ``CFG_TZSRAM_SIZE=0x40000`` assigns SYSRAM only to OP-TEE.
- ``CFG_TZSRAM_SIZE=0x60000`` assigns SYSRAM and SRAM1 to OP-TEE.
- ``CFG_TZSRAM_SIZE=0x80000`` assigns SYSRAM, SRAM1 and SRAM2 to OP-TEE.
- ``CFG_TZSRAM_SIZE=0x90000`` assigns SYSRAM, SRAM1, SRAM2 and SRAM3 to OP-TEE.
- ``CFG_TZSRAM_SIZE=0xa0000`` assigns SYSRAM, SRAM1, SRAM2, SRAM3 and SRAM4 to
  OP-TEE.

Using the eMMC RPMB partition for OP-TEE secure storage
=======================================================

STM32MP15 ED1 and EV1 boards embed an eMMC device that can be used for
OP-TEE secure storage.

Build configuration switch ``WITH_RPMB_TEST=y|n``, when enabled (``y``),
enables ``CFG_RPMB_FS`` and ``CFG_RPMB_TESTKEY`` OP-TEE OS configuration
switches to allow one to test OP-TEE RPMB secure storage support.

See ":ref:`rpmb`" for more information on OP-TEE RPMB secure storage.

Support for EFI secure variables
================================

For platforms embedding an eMMC device with a RPMB partition, one can
use OP-TEE to harden EFI secure boot storing EFI secure variables in OP-TEE
secure storage. This can be used when booting a platform with a UEFI boot
scheme.

Build configuration switch ``WITH_STMM=y|n``, when enabled (``y``),
build and embed EDK2 StMM application in OP-TEE core to secure EFI variables
using OP-TEE RPMB secure storage. ``WITH_STMM=y`` default enables
``WITH_RPMB_TEST``.

.. _STM32MP135F-DK: https://www.st.com/en/evaluation-tools/stm32mp135f-dk.html
.. _STM32MP157A-DK1: https://www.st.com/en/evaluation-tools/stm32mp157a-dk1.html
.. _STM32MP157D-DK1: https://www.st.com/en/evaluation-tools/stm32mp157d-dk1.html
.. _STM32MP157C-DK2: https://www.st.com/en/evaluation-tools/stm32mp157c-dk2.html
.. _STM32MP157F-DK2: https://www.st.com/en/evaluation-tools/stm32mp157f-dk2.html
.. _STM32MP157C-EV1: https://www.st.com/en/evaluation-tools/stm32mp157c-ev1.html
.. _STM32MP157F-EV1: https://www.st.com/en/evaluation-tools/stm32mp157f-ev1.html
.. _Wiki STM32MP135x-DK: https://wiki.st.com/stm32mpu/wiki/STM32MP135x-DK_-_hardware_description
.. _Wiki STM32MP157x-DKx: https://wiki.st.com/stm32mpu/wiki/STM32MP157x-DKx_-_hardware_description
.. _Wiki STM32MP157x-EV1: https://wiki.st.com/stm32mpu/wiki/STM32MP157x-EV1_-_hardware_description
