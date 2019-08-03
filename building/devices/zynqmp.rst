.. _zynqmp:

#########################
ZynqMP zcu10x and Ultra96
#########################
Instructions below show how to run OP-TEE on ZynqMP zcu10x and Ultra96 board.

Supported boards
****************
This makefile supports the following ZynqMP boards:
	
	* zcu102
	* zcu104
	* zcu106
	* Ultra96v1

Setting up the toolchain
************************
This build chain heavily relies on Petalinux 2018.2 therefore the first step is 
to download and install the Petalinux 2018.2 toolchain from the Xilinx website 
(`Downloads`_). Then, you have to download the needed BSP file from the Xilinx 
website (`Downloads`_). You may have to create a free Xilinx account to proceed 
with the two previous steps.

Since OP-TEE 3.6.0, building process relies on `pyelftools`_ and `pycrypto`_ 
package which are not available in the python distribution provided with 
Petalinux, you have to manually install it by following steps described 
hereafter (replace ``/path/to/petalinux`` with the right path):

.. code-block:: bash

	$ sudo apt install python3.5 python3-pip

	$ pip3 install --system --target=/path/to/petalinux/components/yocto/
	source/aarch64/buildtools/sysroots/x86_64-petalinux-linux/usr/lib/
	python3.5/site-packages/ pycrypto

	$ pip3 install --system --target=/path/to/petalinux/components/yocto/
	source/aarch64/buildtools/sysroots/x86_64-petalinux-linux/usr/lib/
	python3.5/site-packages/ pyelftools

Configuring and building for zcu102 board
*****************************************
First, create a new directory which will be used as root directory:

.. code-block:: bash

	$ mkdir -p ~/petalinux-optee
	$ cd ~/petalinux-optee

Then, copy the zcu102 BSP file into the newly created directory:

.. code-block:: bash

	$ cp ~/Downloads/xilinx-zcu102-v2018.2-final.bsp .

Git clone the ``build`` repository of the OP-TEE project and source the 
Petalinux settings:

.. code-block:: bash

	$ git clone https://github.com/OP-TEE/build
	$ cd ./build
	$ source /path/to/petalinux/settings.sh

Finally, use the following commands to create, patch, configure and build the 
Petalinux project. Petalinux is a powerful but very slow tool, each command may 
take a while according to the capabilities of your computer.

.. code-block:: bash
	
	$ make -f zynqmp.mk

Once the last command ends up you are ready to run QEMU tool or to make a 
bootable SD card. To run QEMU:

.. code-block:: bash

	$ make -f zynqmp.mk qemu

QEMU will start and launch Petalinux distribution. At the end of the boot 
process, log in using username ``root`` and password ``root``. Start the OP-TEE 
Normal World service and run xtest:

.. code-block:: bash
	
	$ tee-supplicant -d
	$ xtest
	
You can close QEMU session at any time by typing ``Ctrl-A+C`` and entering the 
``quit`` command.

Configuring and building for other ZynqMP boards
*************************************************
To use this makefile with other supported boards, you have to download the 
corresponding BSP and add option ``PLATFORM`` to each make command.

.. code-block:: bash
	 
	$ make -f zynqmp.mk PLATFORM=zcu106
	$ make -f zynqmp.mk PLATFORM=zcu106 qemu

Hereafter the list of available ``PLATFORM``:

	* ``zcu102``
	* ``zcu104``
	* ``zcu106``
	* ``ultra96-reva``

.. warning::

	On Ultra96 board, UART is not directly available. You have to connect
	through WIFI Access Point using the procedure detailed here 
	`Getting started`_.

SD card creation
****************
After completion of building process, you can create a bootable SD card. Here,
we consider that SD card corresponds to ``/dev/sdb``. We will use ``gparted``
and ``e2image`` tools.

Using ``gparted`` or any other partition manager tool create two partitions on 
the card:

	* 1GB FAT32 bootable partition (``/dev/sdb1`` hereafter).
	* EXT4 partition on the remaining memory space (``/dev/sdb2``
	  hereafter).

Once SD card is partitioned, use the following commands:

.. code-block:: bash
	 
	$ cp /path/to/project/images/linux/BOOT.BIN /dev/sdb1
	$ cp /path/to/project/images/linux/image.ub /dev/sdb1
	$ sudo e2image -rap /path/to/project/images/linux/rootfs.ext4 /dev/sdb2

Now you can use the newly created SD card to boot your board.

.. note::

	Check that your board is actually configured to boot on the SD card.

Building a given version of OP-TEE
**********************************
By default, the lastest version of OP-TEE is built. If you wish you can build a 
given version of OP-TEE instead of the last one by using variable ``OPTEE_VER`` 
with target ``petalinux-config``. See below an example where OP-TEE v3.4.0 is 
built.

.. code-block:: bash
	 
	$ make -f zynqmp.mk petalinux-create
	$ make -f zynqmp.mk OPTEE_VER=3.4.0 petalinux-config
	$ make -f zynqmp.mk petalinux-build

Customizing the Petalinux distribution
**************************************
You can customize the Petalinux project (i.e. kernel, rootfs, ...) as any 
standard Petalinux project. Just enter the project directory and type your 
commands. For additional information, refer to Petalinux Tool Documentation 
(`UG1144`_).

.. _UG1144: https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf
.. _Downloads: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2018-2.html
.. _pyelftools: https://pypi.org/project/pyelftools/
.. _pycrypto: https://pypi.org/project/pycrypto/
.. _Getting started: https://ultra96-pynq.readthedocs.io/en/latest/getting_started.html
