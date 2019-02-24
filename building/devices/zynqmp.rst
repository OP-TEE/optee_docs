.. _zynqmp_zcu102:

#######################
QEMU for ZynqMP zcu102
#######################
Instructions below show how to run OP-TEE using QEMU for ZynqMP zcu102 board. 

Setting up the toolchain
************************
This build chain heavily relies on Petalinux 2018.2 therefore the first step is 
to download and install the Petalinux 2018.2 toolchain from the Xilinx website 
(`Downloads`_). By default, the makefile considers that Petalinux is installed 
in ``/opt/Xilinx/petalinux_2018_2``. Then, you have to download the zcu102 BSP 
file from the Xilinx website (`Downloads`_).

You may have to create a free Xilinx account to proceed with the two previous 
steps.

Configuring and building
************************
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
	$ source /opt/Xilinx/petalinux_2018_2/settings.sh

Finally, use the following commands to create, patch, configure and build the 
Petalinux project. Petalinux is a powerful but very slow tool, each command may 
take a while according to the capabilities of your computer.

.. code-block:: bash
	
	$ make -f zynqmp.mk petalinux-create
	$ make -f zynqmp.mk petalinux-config
	$ make -f zynqmp.mk petalinux-build

Once the last command ends up you are ready to run QEMU tool. Use the following 
command:

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

QEMU for other ZynqMP boards
******************************
This makefile supports other ZynqMP boards:
	
	* zcu104
	* zcu106

To use this makefile with these boards, you have to download corresponding BSP 
and add option ``PLATFORM=zcu104`` or ``PLATFORM=zcu106`` to each make command.

.. code-block:: bash
	 
	$ make -f zynqmp.mk PLATFORM=zcu=106 petalinux-create
	$ make -f zynqmp.mk PLATFORM=zcu=106 petalinux-config
	$ make -f zynqmp.mk PLATFORM=zcu=106 petalinux-build
	$ make -f zynqmp.mk PLATFORM=zcu=106 qemu

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
