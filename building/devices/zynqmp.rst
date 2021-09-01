.. _zynqmp:

##########
Zynq MPSoC
##########
Instructions below show how to run OP-TEE on Zynq MPSoC based boards.

Supported boards
*****************************************
+------------+--------------+----------------------+
| Board Name | Manufacturer | Hardware Description |
+============+==============+======================+
| ZCU102     | Xilinx/AMD   | `ZCU102 Website`_    |
+------------+--------------+----------------------+
| ZCU104     | Xilinx/AMD   | `ZCU104 Website`_    |
+------------+--------------+----------------------+
| ZCU106     | Xilinx/AMD   | `ZCU106 Website`_    |
+------------+--------------+----------------------+
| Ultra96    | Avnet        | `Ultra96 Website`_   |
+------------+--------------+----------------------+

Boot Firmware
*****************************************
Xilinx Zynq MPSoC device requires two firmware images, one to configure the
device (First Stage Bootloader) and one for runtime platform management (PMU
Firmware). The scope of OP-TEE build Makefile does not cover buildling these two
firmware images therefore pre built binaries are required to generate a valid
boot image. The pre built images can be found in the following `Xilinx wiki
<https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842316/Linux+Prebuilt+Images>`_
page.

.. note::
	For Ultra96 board, the firmware binaries can be found in the Avnet website.

Build instructions
*****************************************
Follow the instructions at ":ref:`get_and_build_the_solution`" page.

Configuration switch ``PLATFORM`` can be used to specify the target device
as listed in table below:

+------------+-------------------------------+
| Board Name | Build configuration directive |
+============+===============================+
| ZCU102     | ``PLATFORM=zynqmp-zcu102``    |
+------------+-------------------------------+
| ZCU104     | ``PLATFORM=zynqmp-zcu104``    |
+------------+-------------------------------+
| ZCU106     | ``PLATFORM=zynqmp-zcu106``    |
+------------+-------------------------------+
| Ultra96    | ``PLATFORM=zynqmp-ultra96``   |
+------------+-------------------------------+

An example of fetch and build commands is:

.. code-block:: bash

  $ repo init -u https://github.com/OP-TEE/manifest.git -m zynqmp.xml
  $ repo sync
  $ cd build
  $ make toolchains
  $ make PLATFORM=zynqmp-zcu102 all

After completion of the buildling process, two new files will be generated
within the ``zynqmp/`` folder, ``BOOT.bin`` and ``<platform-name>.ub``. The
first one is the boot image composed of the FSBL, PMU Firmware, ARM Trusted
Firmware, OP-TEE and U-Boot. The second one is a FIT image containing the Linux
kernel, the device-tree blob and the initramfs root file system.

.. note::
	If the firmware image is not provided to the build script the boot image
	will not be generated.

Petalinux build instructions
*****************************************
OP-TEE build can be additionally integrated within Xilinx Petalinux tool for
Embedded Linux development. As Petalinux is built on top of Yocto, the
integration is performed through adding some exisiting recipes and few
customizations. Use the previous build `Makefile
<https://github.com/OP-TEE/build/blob/3.14.0/zynqmp.mk>`_ based on Petalinux
2020.2 release as reference.

Booting the device
*****************************************
SD Card boot
=============
Place both generated images in a single partition within the SD card. Boot the
board in SD boot mode and stop the U-Boot autoboot process once the prompt is
displayed in the serial port.

Use the bellow commands to load the FIT image to RAM and boot.

.. code-block:: none

	ZynqMP> fatload mmc 0 0x30000000 zynqmp-zcu102.ub
	27803872 bytes read in 1827 ms (14.5 MiB/s)
	ZynqMP> bootm 0x30000000

.. _ZCU102 Website: https://www.xilinx.com/products/boards-and-kits/ek-u1-zcu102-g.html
.. _ZCU104 Website: https://www.xilinx.com/products/boards-and-kits/zcu104.html
.. _ZCU106 Website: https://www.xilinx.com/products/boards-and-kits/zcu106.html
.. _Ultra96 Website: https://www.96boards.org/product/ultra96/
