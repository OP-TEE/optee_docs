.. _stmm:

############
StandAloneMM
############

StandAlomeMM is a PE/COFF binary produced by EDK2. For Arm platforms we 
can compile and use it, in combination with OP-TEE to store EFI variables
in and RPMB partition of our eMMC.

EDK2 Build instructions
***********************

.. code-block:: bash

    $ git clone https://github.com/tianocore/edk2.git
    $ git clone https://github.com/tianocore/edk2-platforms.git
    $ cd edk2
    $ git submodule init && git submodule update --init --recursive
    $ cd ..
    $ export WORKSPACE=$(pwd)
    $ export PACKAGES_PATH=$WORKSPACE/edk2:$WORKSPACE/edk2-platforms
    $ export ACTIVE_PLATFORM="Platform/StandaloneMm/PlatformStandaloneMmPkg/PlatformStandaloneMmRpmb.dsc"
    $ export GCC5_AARCH64_PREFIX=aarch64-linux-gnu-
    $ source edk2/edksetup.sh
    $ make -C edk2/BaseTools
    $ build -p $ACTIVE_PLATFORM -b RELEASE -a AARCH64 -t GCC5 -n `nproc`

OP-TEE Build instructions
*************************

.. code-block:: bash
    
    $ git clone https://github.com/OP-TEE/optee_os.git
    $ cd optee_os
    $ ln -s ../Build/MmStandaloneRpmb/RELEASE_GCC5/FV/BL32_AP_MM.fd
    $ export ARCH=arm
    $ CROSS_COMPILE32=arm-linux-gnueabihf- make -j32 CFG_ARM64_core=y \
        PLATFORM=<myboard> CFG_STMM_PATH=BL32_AP_MM.fd CFG_RPMB_FS=y \
        CFG_RPMB_FS_DEV_ID=0 CFG_CORE_HEAP_SIZE=524288 CFG_RPMB_WRITE_KEY=1 \
        CFG_CORE_HEAP_SIZE=524288 CFG_CORE_DYN_SHM=y CFG_RPMB_TESTKEY=y \
        CFG_REE_FS=n CFG_CORE_ARM64_PA_BITS=48  CFG_TEE_CORE_LOG_LEVEL=1 \
        CFG_TEE_TA_LOG_LEVEL=1 CFG_SCTLR_ALIGNMENT_CHECK=n

U-Boot Build instructions
*************************

Although the StandAloneMM binary comes from EDK2, using and storing the
variables is currently available in U-Boot only.

.. code-block:: bash
    
    $ git clone https://github.com/u-boot/u-boot.git
    $ cd u-boot
    $ export CROSS_COMPILE=aarch64-linux-gnu-
    $ export ARCH=<arch>
    $ make <myboard>_defconfig
    $ make menuconfig

Enable ``CONFIG_OPTEE``, ``CONFIG_CMD_OPTEE_RPMB`` and ``CONFIG_EFI_MM_COMM_TEE``

.. code-block:: bash
    
    $ make -j `nproc`


.. warning::
    
    - Your OP-TEE platform port must support Dynamic shared memory, since that's
      the only kind of memory U-Boot supports for now.
