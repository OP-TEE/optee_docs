.. _file_structure:

File structure
##############
This page describes organization of the tree structure in :ref:`optee_os`.

The description is dived into different tables. First the flat top directory
followed by the ``core/`` directory tree with the ``core/arch/arm/`` tree
in separate table. There are two more tables covering the ``lib/`` and
``ta/`` trees.

Top level directories
*********************
.. list-table::
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - core/
      - Files that are only used building OP-TEE core, the privileged mode part

    * - keys/
      - Secure keys or not so secure example keys

    * - ldelf/
      - Ldelf the user mode ELF loader, for instance used to load TAs

    * - lib/
      - Libraries that are used both when building more than one component,
        for instance, OP-TEE core, ldelf, or TAs

    * - mk/
      - Makefiles supporting the build system

    * - scripts/
      - Helper scripts for miscellaneous tasks

    * - ta/
      - Files that are only used when building TAs

    * - out/
      - Created when building unless a different out directory is specified with
        ``O=...`` on the command line

core/
*****
.. list-table::
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - arch/
      - Architecture and platform specific files

    * - arch/arm/
      - Arm specific architecture and platform files

    * - crypto/
      - Crypto infrastructure including software implementations of certain
        algorithms.

    * - drivers/
      - Various device drivers

    * - include/
      - Header files of resources exported to the rest of the core

    * - include/crypto/
      - Include files related to files in /core/crypto

    * - include/drivers/
      - Include files related to device drivers

    * - include/dt-bindings/
      - Include files for the device tree bindings

    * - include/kernel/
      - Include files related to files in /core/kernel

    * - include/mm/
      - Include files related to memory management and files in /core/mm

    * - include/tee/
      - Include files related to files in /core/tee

    * - kernel/
      - Miscellaneous architecture neutral files

    * - lib/
      - Libraries that are used by core only

    * - lib/libfdt/
      - Flat Device Trees manipulation library

    * - lib/libfdt/include/
      - Include files related to libfdt

    * - lib/libtomcrypt/
      - Libtomcrypt crypto library

    * - lib/libtomcrypt/include/
      - Include files related to libtomcrypt

    * - lib/libtomcrypt/src/
      - Source files of libtomcrypt

    * - lib/zlib/
      - Zlib compression library

    * - mm/
      - Architecture neutral memory management

    * - pta/
      - Various pseudo TAs

    * - tee/
      - Architecture neutral TEE files

core/arch/arm/
**************
.. list-table::
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - cpu/
      - CPU specific settings

    * - crypto/
      - Architecture specific software implementations of crypto algorithms

    * - dts/
      - Device tree source files

    * - include/
      - Header files of resources exported to the rest of the core

    * - include/crypto/
      - Architecture specific include files related to /core/crypto or
        /core/arch/arm/crypto files

    * - include/kernel/
      - Architecture specific include files related to /core/kernel or
        /core/arch/arm/kernel files

    * - include/mm/
      - Architecture specific include files related to /core/mm or
        /core/arch/arm/mm files

    * - include/sm/
      - Include files related to the secure monitor

    * - include/tee/
      - Architecture specific include files related to /core/tee or
        /core/arch/arm/tee files

    * - kernel/
      - Miscellaneous low level architecure specific files

    * - plat-\*/
      - Specific files for the different supported platform

    * - mm/
      - Memory management

    * - tee/
      - TEE files

    * - sm/
      - Secure Monitor, ARMv7-A only

lib/
*************
.. list-table::
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - libdl/
      - Implementation of dlopen(), dlsym() and dlclose() used by TAs and ldelf

    * - libdl/include/
      - Include files for libdl

    * - libmbedtls/
      - Mbed TLS crypto library

    * - libmbedtls/core/
      - Glue code only compiled with core to connect with the core internal
        <crypto/crypto.h> API.

    * - libmbedtls/include/
      - Include files with configuration of Mbed TLS

    * - libmbedtls/mbedtls/
      - Top directory of the imported Mbed TLS source tree

    * - libmbedtls/mbedtls/include/
      - Mbed TLS include files

    * - libmbedtls/mbedtls/library/
      - Mbed TLS implementation

    * - libunw/
      - Unwind library 

    * - libunw/include/
      - Include files for libunwnd

    * - libutee/
      - Libutee which provide the implementation of TEE Internal Core API.

    * - libutee/arch/
      - Architecture specific implementation

    * - libutee/include/
      - Include files related to libutee and the header files for
        TEE Internal Core API

    * - libutils/
      - The reduced "libc" of OP-TEE

    * - libutils/ext/
      - Extensions to a standard libc

    * - libutils/ext/arch/
      - Architecture specific implmementation of the extensions

    * - libutils/ext/include/
      - Include files related to the extensions

    * - libutils/isoc/
      - A subset of ISOC

    * - libutils/isoc/arch/
      - Architecture specific

    * - libutils/isoc/include/
      - Header files related to the provided subset of ISOC

    * - libutils/isoc/newlib/
      - Routines imported from newlib


ta/
*************
.. list-table::
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - trusted_keys
      - Trusted key TA

    * - trusted_keys/include
      - Header file of the ABI provided by the trusted key TA

    * - arch
      - Architecture specific files needed to compile a TA

    * - mk
      - Makefile includes needed to build TAs and the TA dev kit

    * - avb
      - TA to support AVB (Android Verified Boot)

    * - avb/include
      - Header file of the ABI provided by the AVB TA

    * - pkcs11
      - TA to support PKCS#11 

    * - pkcs11/src
      - Source code for the PKCS#11 TA

    * - pkcs11/include
      - Header file for the ABI provided by the PKCS#11 TA
