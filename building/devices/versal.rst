.. _versal:

#############################
AMD/Xilinx Versal ACAP VCK190
#############################
Instructions below show how to run OP-TEE on VCK190.

Supported boards
****************
This makefile supports the VCK190 but can be extended to support VCK180 as well.

Setting up the toolchain
************************
This build chain relies on Petalinux 2022.1, therefore the first step will be to
download and install it from the Xilinx website (`Downloads`_).

Then, you will also need to download the board support package (BSP) from the
Xilinx website (`Downloads`_). It contains prebuilt firmwares and hardware
definition files required to assemble a bootable image.

.. note::
   You may have to create a free Xilinx account to proceed with the two previous steps.

Configuring and building for VCK190
***********************************
Lets summarize the steps taken so far; these are common to all boards.

.. code-block:: bash

	$ mkdir ~/optee-project
	$ cd ~/optee-project
	$ repo init -u https://github.com/OP-TEE/manifest.git -m versal.xml
	$ repo sync -j4 --no-clone-bundle
	$ cd build
	$ make -j8 toolchains
	$ make -j8

At this point we have a working directory ``~/optee-project`` with all the
repositories required with the exception of the Versal ACAP board support
package. A pre-requisite to unpacking the BSP file is an installed Petalinux
release as previously mentioned.

Having done that, now is the time to unpack the BSP:

.. code-block:: bash

	$ cd ~/optee-project
	$ cp ~/Downloads/xilinx-vck190-v2022.1-04191534.bsp .
	$ source /path/to/petalinux.2022.1/settings.sh
	$ petalinux-create --type project -s xilinx-vck190-v2022.1-04191534.bsp
	$ ls
	   xilinx-vck190-2022.1


Before building the release you will need to edit the file
``build/versal/bootImage-versal-vck190.bif`` to point to the BSP folder.

After you have done that you can build the images as follows:

.. code-block:: bash

	$ cd ~/optee-project
	$ cd build
	$ make -f versal.mk image
	$ ls versal | grep -E 'BIN|ub'
	  BOOT.BIN
	  versal-vck190.ub


JTAG boot to U-Boot shell
*************************
To run the bootable image ``BOOT.BIN`` via JTAG, configure the boot switches as
seen below and then power up the board.

.. figure:: /images/boards/vck190-jtag-boot.png
	:width: 400
	:align: center

Then run the boot_jtag.sh script.

This script will first ask for the path of the Petalinux installation; once
entered, if will download and execute the image on the Versal ACAP platform.

.. code-block:: bash

	$ cd ~/optee-project/build/versal/
	$ ./boot_jtag.sh



SD card creation and boot
*************************
Prepare a SD card with a single **bootable** partition large enough to hold both of
the built files.

Using ``gparted`` or any other partition manager tool create a single partition on
the card (remember to flag it as bootable)

	* 1GB FAT32 bootable partition (i.e: ``/dev/sdc1``).

Once SD card is partitioned, mount it on your file system and copy the images:

.. code-block:: bash

	$ cp ~/optee-project/build/versal/BOOT.BIN <mount_point>/
	$ cp ~/optee-project/build/versal/versal-vck190.ub <mount_point>/
	$ sync
	$ umount <mount_point>

Now you can use the newly created SD card to boot your board. Make sure the boot
switches are configured for SD boot.

.. figure:: /images/boards/vck190-sd-boot.png
	:width: 400
	:align: center

Unless you have modified the default U-boot boot command, you will need to stop
the sequence at the U-boot shell and issue these three additional commands to
boot to Linux:

.. code-block:: bash

	uboot shell$ mmc dev 0
	uboot shell$ fatload mmc 0:1 0x20000000 versal-vck190.ub
	uboot shell$ bootm 0x20000000


.. _Downloads: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2022-1.html
