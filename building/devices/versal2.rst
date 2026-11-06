.. _versal2:

################
AMD Versal Gen 2
################

Instructions below show how to build and run OP-TEE OS for the AMD Versal Gen
2 platform.


Prerequisites
*************

Install prerequisites according to the `OP-TEE prerequisites page
<https://optee.readthedocs.io/en/latest/building/prerequisites.html>`_.


Configuring and building
*************************

Fetching source code
=====================

.. code-block:: bash

    # Create a new project directory
    mkdir -p optee_project
    cd optee_project
    # Initialize the repo with the remote manifest file from Upstream repo
    repo init -u https://github.com/OP-TEE/manifest -m versal2.xml
    # Sync all the required repos
    repo sync

Preparing toolchains
====================

.. code-block:: bash

    # Enter the repo build directory
    cd build
    # Get all the required tool chains
    make toolchains

This shall have created a ``toolchains`` directory within the
``optee_project/`` working directory with all the required cross-compile
toolchain executables.

Building the artifacts
=======================

.. code-block:: bash

    # This is required ONLY if compiling through docker
    export FORCE_UNSAFE_CONFIGURE=1
    # Start compilation
    make all

This builds every component required for the AMD Versal Gen 2 platform -
ARM Trusted Firmware, OP-TEE OS, U-Boot, Linux kernel, device tree
overlay/blobs, Buildroot root filesystem, ``bootgen``, QEMU and the QEMU
device trees - and finally assembles the bootable image. All resulting
artifacts are collected under ``optee_project/out/bin``.


Execution on QEMU
*****************

Once ``make all`` has completed successfully, the platform can be booted
directly on QEMU:

.. code-block:: bash

    # From optee_project/build/
    make run-only

AMD Versal Gen 2 models three independent processing units, so this target
launches three QEMU instances at once:

- **MicroBlaze PMC** - runs the PLM firmware, started in the background.
- **RISC-V ASU** - runs the ASU firmware, started in the background.
- **AArch64 PSXC** - the main application processor running ARM Trusted
  Firmware, OP-TEE OS, U-Boot and Linux, started interactively in the
  foreground.

Console output of the MicroBlaze and RISC-V instances is redirected to
``mb-pmc.log`` and ``riscv-asu.log`` respectively under
``optee_project/out/bin``, while the AArch64 instance's console is attached
to the current terminal. Use ``Ctrl+a x`` to exit the interactive AArch64
QEMU instance; the background MicroBlaze and RISC-V instances are then
stopped automatically.

.. note::

    ``make run`` can be used instead of ``make run-only`` to build all
    artifacts (equivalent to ``make all``) and then boot QEMU in a single
    step.

Booting Linux from U-Boot
==========================

The AArch64 QEMU instance loads the Linux ``Image`` and the root filesystem
ramdisk directly into memory at fixed addresses. Stop at the U-Boot prompt and
boot Linux with the ``booti`` command, passing the addresses of the kernel
image, the ramdisk and the control FDT already loaded by U-Boot:

.. code-block:: console

    versal2> booti 0x21000000 0x30000000 $fdtcontroladdr
