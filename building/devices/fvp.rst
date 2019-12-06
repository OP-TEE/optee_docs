.. _fvp:

###
FVP
###
The instructions here will tell how to build and run OP-TEE using Foundation
Models.


.. _fvp_build_instructions:

Build instructions
******************
Start out by following the ":ref:`get_and_build_the_solution`" as described in
:ref:`build`. However, stop before doing ":ref:`build_make`".

Next you should obtain the `Armv8-A Foundation Platform (For Linux Hosts
Only)`_. To download FVPs you’ll need to log in to Arm Self Service. That binary
should be untar'ed to the root of the repo forest, i.e., like this:
``<fpv-project>/Foundation_Platformpkg``. In the end after cloning all source
code, getting the toolchains and "installing" Foundation_Platformpkg you should
have a folder structure that looks like this:

.. code-block:: none
    :emphasize-lines: 9

    $ ls -al
    drwxrwxr-x 15 jbech jbech 4096 Feb  5 09:10 .
    drwxr-xr-x 22 jbech jbech 4096 Jan 15 12:45 ..
    drwxrwxr-x 18 jbech jbech 4096 Feb  5 09:10 arm-trusted-firmware
    drwxrwxr-x  9 jbech jbech 4096 Feb  5 09:10 build
    drwxrwxr-x 15 jbech jbech 4096 Feb  5 09:10 buildroot
    drwxrwxr-x 51 jbech jbech 4096 Feb  5 09:10 edk2
    drwxrwxr-x  5 jbech jbech 4096 Feb  5 09:10 edk2-platforms
    drwxrwxr-x  6 jbech jbech 4096 Mar 15  2018 Foundation_Platformpkg
    drwxrwxr-x 15 jbech jbech 4096 Feb  5 09:10 grub
    drwxrwxr-x 26 jbech jbech 4096 Feb  5 09:10 linux
    drwxrwxr-x  6 jbech jbech 4096 Feb  5 09:10 optee_client
    drwxrwxr-x 10 jbech jbech 4096 Feb  5 09:10 optee_examples
    drwxrwxr-x 11 jbech jbech 4096 Feb  5 09:10 optee_os
    drwxrwxr-x  8 jbech jbech 4096 Feb  5 09:10 optee_test
    drwxrwxr-x  7 jbech jbech 4096 Feb  5 09:10 .repo
    lrwxrwxrwx  1 jbech jbech   23 Feb  5 09:09 toolchains

When this pre-condition met you can simply continue with

.. code-block:: bash

    $ make run

and then FVP should build the rootfs and then start the simulation and when you
have a terminal you can log in and run xtest (as described at
:ref:`build_run_xtest`).

.. _change_fVP_foundation_model_to_base_model:

Change FVP Foundation Model to Base Model
******************
If you need to run OP-TEE on FVP Base Model, you can make the following modifications after **completing FVP Foundation Model configuration and successfully running xtest.**

.. _step_download_fvp_base_model:

Step 1 - Download ARMv8-A Base Platform Binary (Lin) FOC
============================
* Download `ARMv8-A Base Platform Binary (Lin) FOC <https://silver.arm.com/browse/FM000/>`_ with GCC 4.9 version (not the latest one), unpack this file to get a **Base_RevC_AEMv8A_pkg** folder.
* Replace the **Foundation_Platformpkg** folder with  **Base_RevC_AEMv8A_pkg** folder.
* Change FVP Path in  **fvp.mk** to Base_RevC_AEMv8A_pkg

.. code-block:: makefile

    #FOUNDATION_PATH		?= $(ROOT)/Foundation_Platformpkg
    FOUNDATION_PATH		?= $(ROOT)/Base_RevC_AEMv8A_pkg

.. _step_change_dtb_file:

Step 2 - Change the dtb file
============================
* Add following child node to the root node in **linux/arch/arm64/boot/dts/arm/fvp-base-revc.dts** file.

.. code-block:: 

    firmware {
            optee {
                    compatible = "linaro,optee-tz";
                    method = "smc";
            };
    }; 

* Modify the makefile **fvp.mk**.

.. code-block:: makefile

    ################################################################################
    # Boot Image
    ################################################################################
    .PHONY: boot-img
    boot-img: linux grub buildroot
        rm -f $(BOOT_IMG)
        mformat -i $(BOOT_IMG) -n 64 -h 255 -T 131072 -v "BOOT IMG" -C ::
        mcopy -i $(BOOT_IMG) $(LINUX_PATH)/arch/arm64/boot/Image ::
        #mcopy -i $(BOOT_IMG) $(LINUX_PATH)/arch/arm64/boot/dts/arm/foundation-v8-gicv3-psci.dtb ::
        mcopy -i $(BOOT_IMG) $(LINUX_PATH)/arch/arm64/boot/dts/arm/fvp-base-revc.dtb ::
        mmd -i $(BOOT_IMG) ::/EFI
        mmd -i $(BOOT_IMG) ::/EFI/BOOT
        mcopy -i $(BOOT_IMG) $(ROOT)/out-br/images/rootfs.cpio.gz ::/initrd.img
        mcopy -i $(BOOT_IMG) $(GRUB_BIN) ::/EFI/BOOT/bootaa64.efi
        mcopy -i $(BOOT_IMG) $(GRUB_CONFIG_PATH)/grub.cfg ::/EFI/BOOT/grub.cfg

* Modify **grub.cfg**.

.. code-block::

    set prefix='/EFI/BOOT'

    set default="0"
    set timeout=10

    menuentry 'GNU/Linux (OP-TEE)' {
        linux /Image console=tty0 console=ttyAMA0,115200 earlycon=pl011,0x1c090000 root=/dev/disk/by-partlabel/system rootwait rw ignore_loglevel efi=noruntime
        initrd /initrd.img
        devicetree /fvp-base-revc.dtb
    }

.. _step_change_fvp_start_params:

Step 3 - Change FVP start parameters
============================
* Modify the makefile **fvp.mk**.

.. code-block:: makefile

    run-only:
        @cd $(FOUNDATION_PATH); \
        $(FOUNDATION_PATH)/models/Linux64_GCC-4.9/FVP_Base_RevC-2xAEMv8A \
        -C pctl.startup=0.0.0.0 		\
        -C bp.secure_memory=1			\
        -C bp.tzc_400.diagnostics=1  		\
        -C cluster0.NUM_CORES=4 				\
        -C cluster1.NUM_CORES=4 				\
        -C cache_state_modelled=1 				\
        --data "$(TF_A_PATH)/build/fvp/$(TF_A_BUILD)/bl1.bin"@0x0 		\
        --data "$(TF_A_PATH)/build/fvp/$(TF_A_BUILD)/fip.bin"@0x8000000 	\
        -C bp.virtioblockdevice.image_path="$(BOOT_IMG)"

.. _step_make_clean_make_run:

Step 4 - Make clean & make run
============================
If following error occurs after executing "make run" command,

.. code-block::

    FVP_Base_RevC-2xAEMv8A: '/home/optee/fvpoptee/build/../trusted-firmware-a/build/fvp//bl1.bin': data file not found

check the **TF_A_BUILD** variable in **fvp.mk** and make sure the first few lines of it are as follows.

.. code-block:: makefile

    ################################################################################
    # Following variables defines how the NS_USER (Non Secure User - Client
    # Application), NS_KERNEL (Non Secure Kernel), S_KERNEL (Secure Kernel) and
    # S_USER (Secure User - TA) are compiled
    ################################################################################
    COMPILE_NS_USER   ?= 64
    override COMPILE_NS_KERNEL := 64
    COMPILE_S_USER    ?= 64
    COMPILE_S_KERNEL  ?= 64

    include common.mk


    ################################################################################
    # Paths to git projects and various binaries
    ################################################################################
    TF_A_PATH		?= $(ROOT)/trusted-firmware-a
    ifeq ($(DEBUG),1)
    TF_A_BUILD		?= debug
    else
    TF_A_BUILD		?= release
    endif
    EDK2_PATH		?= $(ROOT)/edk2
    EDK2_PLATFORMS_PATH	?= $(ROOT)/edk2-platforms
    EDK2_BIN		?= $(EDK2_PLATFORMS_PATH)/Build/ArmVExpress-FVP-AArch64/RELEASE_GCC49/FV/FVP_AARCH64_EFI.fd
    FOUNDATION_PATH		?= $(ROOT)/Base_RevC_AEMv8A_pkg

    ...


.. _Armv8-A Foundation Platform (For Linux Hosts Only): https://developer.arm.com/products/system-design/fixed-virtual-platforms
