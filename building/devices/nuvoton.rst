.. _nuvoton:

#######
NUVOTON
#######

The instructions here will tell how to build and run OP-TEE OS
for Nuvoton platform standalone (not as a part of openbmc image).


.. _nuvoton_build_instructions:

Build instructions
******************

=================
Pre-requirements:
=================

#. Install prerequisites according to the :ref:`prerequisites` page.`

#. Download the latest IGPS from the Nuvoton Israel GitHub repository

    .. code-block:: bash

        $ git clone https://github.com/Nuvoton-Israel/igps-npcm8xx

#. Download and extract the latest Linux based Arm GNU Toolchain
for aarch64 bare-metal target, for example, 12.2

    .. code-block:: bash

        $ cd /opt
        $ wget https://developer.arm.com/-/media/Files/downloads/gnu/12.2.rel1/binrel/arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-elf.tar.xz?rev=28d5199f6db34e5980aae1062e5a6703&hash=D87D4B558F0A2247B255BA15C32A94A9F354E6A8
        $ tar xvf arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-elf.tar.xz

#. Add the Arm GNU Toolchain binary to the $PATH

    .. code-block:: bash

        $ cd ~
        $ vi .bashrc
        export PATH=$PATH:/opt/arm-gnu-toolchain-12.2.rel1-x86_64-aarch64-none-elf/bin

#. Clone the latest OP-TEE OS code from the GitHub repository

    .. code-block:: bash

        $ git clone https://github.com/OP-TEE/optee_os.git
        $ cd optee_os

==============
Build process:
==============

#. build OP-TEE OS for Nuvoton platform

    .. code-block:: bash

        $ make CROSS_COMPILE64=aarch64-none-elf- PLATFORM=nuvoton -j $(nproc)

   Note: you can use additional debug flag for compilation to get debug prints to console

    .. code-block:: bash

        $ make CROSS_COMPILE64=aarch64-none-elf- PLATFORM=nuvoton CFG_NPCM_DEBUG=y -j $(nproc)

#. Update binary input files in IGPS

    .. code-block:: bash

        $ cd ../igps-npcm8xx/py_scripts
        $ python ./UpdateInputsBinaries_Arbel_A1_EB.py

#. Copy the compiled out/arm-plat-nuvoton/core/tee.bin file into IGPS_3.8.6/py_scripts/ImageGeneration/inputs

#. Generate new image file

    .. code-block:: bash

        $ python ./GenerateAll.py

#. Program the new image to flash:

    .. code-block:: bash

        $ python ./ProgramAll_Secure.py

#. After programming, enable terminal connection to the ArbelEVB,
   and if you compiled with the ``CFG_NPCM_DEBUG=y`` flag,
   you will see OP-TEE version before U-Boot console trace messages,
   for example:

   I/TC: >================================================
   I/TC: OP-TEE OS Version 3.21.0-1127-gaf809d0ab-dev (gcc version 12.2 (Arm GNU Toolchain 12.2.Rel1)) #1 Thu May 18 05:24:18 UTC 2023 aarch64
   I/TC: >================================================
