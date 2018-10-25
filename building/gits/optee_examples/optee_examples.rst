.. _optee_examples:

##############
optee_examples
##############
This document describes the sample applications that are included in the OP-TEE,
that aim to showcase specific functionality and use cases.

For sake of simplicity, all OP-TEE example test application are prefixed with
``optee_example_``. All of them works as standalone host and Trusted Application
and can be found in separate directories.

git location
************
https://github.com/linaro-swg/optee_examples

License
*******
.. todo::

    Joakim: Necessary to state that here? Changing the "License headers" page to
    instead become a "License" page and add addtional sections.

The software is provided under the `BSD 2-Clause`_ license.

Build instructions
******************
You can build the code in this git only or build it as part of the entire
system, i.e. as a part of a full OP-TEE developer setup. For the latter, please
refer to instructions at the :ref:`build` page. For standalone builds we
currently support building with both CMake as well as with regular GNU
Makefiles. However, since the both the host and the Trusted Applications have
dependencies to files in :ref:`optee_client` (libteec.so and headers) as well as
:ref:`optee_os` (TA-devkit), one **must first** build those and then refer to
various files. Below we will show to to build the **hello_world** example for
Armv7-A using regular GNU Make.

Configure the toolchain
=======================
First step is to download and configure a toolchain, see the :ref:`toolchains`
page for instructions.

Build the dependencies
======================
Then you must build :ref:`optee_os` as well as :ref:`optee_client` first. Build
instructions for them can be found on their respective pages.

Clone optee_examples
====================
.. code-block:: bash

    $ git clone https://github.com/linaro-swg/optee_examples.git

.. todo::

    Joakim: We should add ...
    Build using CMake
    =================

    But that is not really straight forward to do.

Build using GNU Make
====================

Host application
----------------
.. code-block:: bash

    $ cd optee_examples/hello_world/host
    $ make \
        CROSS_COMPILE=arm-linux-gnueabihf- \
        TEEC_EXPORT=<optee_client>/out/export \
        --no-builtin-variables

With this you end up with a binary ``optee_example_hello_world`` in the host
folder where you did the build.

Trusted Application
-------------------

.. code-block:: bash

    $ cd optee_examples/hello_world/ta
    $ make \
        CROSS_COMPILE=arm-linux-gnueabihf- \
        PLATFORM=vexpress-qemu_virt \
        TA_DEV_KIT_DIR=<optee_os>/out/arm/export-ta_arm32

With this you end up with a files named ``uuid.{ta,elf,dmp,map}`` etc in the ta
folder where you did the build.

.. note::

    For a 64-bit builds (or any other toolchain) you will need to change
    ``CROSS_COMPILE`` (and also use a ``PLATFORM`` corresponding to an Armv8-A
    configuration).


Coding standards
****************
See :ref:`coding_standards`.


Example applications
********************

acipher
=======

    ================================ ========================================
    Application name                 UUID
    ================================ ========================================
    ``optee_example_acipher``        ``a734eed9-d6a1-4244-aa50-7c99719e7b7b``
    ================================ ========================================

Generates an RSA key pair of specified size and encrypts a supplied string with
it using the GlobalPlatform TEE Internal Core API.

aes
===

    ================================ ========================================
    Application name                 UUID
    ================================ ========================================
    ``optee_example_aes``            ``5dbac793-f574-4871-8ad3-04331ec17f24``
    ================================ ========================================

Runs an AES encryption and decryption from a TA using the GlobalPlatform TEE
Internal Core API. Non secure test application provides the key, initial vector
and ciphered data.

.. _hello_world:

hello_world
===========

    ================================ ========================================
    Application name                 UUID
    ================================ ========================================
    ``optee_example_hello_world``    ``8aaaf200-2450-11e4-abe2-0002a5d5c51b``
    ================================ ========================================

This is a very simple Trusted Application to answer a hello command and
incrementing an integer value.

hotp
====

    ================================ ========================================
    Application name                 UUID
    ================================ ========================================
    ``optee_example_hotp``           ``484d4143-2d53-4841-3120-4a6f636b6542``
    ================================ ========================================

.. include:: hotp.rst

random
======

    ================================ ========================================
    Application name                 UUID
    ================================ ========================================
    ``optee_example_random``         ``b6c53aba-9669-4668-a7f2-205629d00f86``
    ================================ ========================================

Generates a random UUID using capabilities of TEE API
(``TEE_GenerateRandom()``).

secure_storage
==============

    ================================ ========================================
    Application name                 UUID
    ================================ ========================================
    ``optee_example_secure_storage`` ``f4e750bb-1437-4fbf-8785-8d3580c34994``
    ================================ ========================================

A Trusted Application to read/write raw data into the OP-TEE secure storage
using the GlobalPlatform TEE Internal Core API.

Further reading
***************
Some additional information about how to write and compile Trusted Applications
can be found at the :ref:`build_trusted_applications` page.

.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
