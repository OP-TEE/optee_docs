.. _optee_client:

############
optee_client
############
optee_client git contains the source code for the TEE client library in Linux.
This component provides the TEE Client API as defined by the GlobalPlatform TEE
standard. It is distributed under the BSD 2-clause open source license.

In this git there are two main targets/binaries to build. There is
``libteec.so``, which is the library that contains that API for communication
with the Trusted OS. Then there is ``tee-supplicant`` which is a daemon serving
the Trusted OS in secure world with miscellaneous features, such as file system
access.

git location
************
https://github.com/OP-TEE/optee_client

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
Makefiles.

Configure the toolchain
=======================
First step is to download and configure a toolchain, see the :ref:`toolchains`
page for instructions.

Clone optee_client
==================
.. code-block:: bash

    $ git clone https://github.com/OP-TEE/optee_client
    $ cd optee_client

Build using CMake
=================
.. code-block:: bash

    $ mkdir build
    $ cd build
    $ cmake -DCMAKE_C_COMPILER=arm-linux-gnueabihf-gcc ..
    $ make

.. note::

    This example uses the 32-bit toolchain (arm-linux-gnueabihf-), the same
    works using the 64-bit toolchain (aarch64-linux-gnu-).

After this step the compiled binaries can be sound in sub-folders of ``build``.
If you have a need or preference to install the binaries at some specific
location, then on the cmake line above add
``-DCMAKE_INSTALL_PREFIX=<my-install-path>`` as an additional argument. With
that you can then run ``make install`` and the binaries etc will be copied to
the location that you gave as an argument. In this example
``/tmp/optee_client``.

.. code-block:: bash

    $ cmake -DCMAKE_C_COMPILER=arm-linux-gnueabihf-gcc -DCMAKE_INSTALL_PREFIX=/tmp/optee_client ..
    $ make
    $ make install

Build using GNU Make
====================
The ``Makefile`` is configured to use ``arm-linux-gnueabihf-`` by default.

.. code-block:: bash

    $ make

.. note::

    For a 64-bit builds (or any other toolchain) you will need to use
    ``CROSS_COMPILE``.

        ``$ make CROSS_COMPILE=aarch64-linux-gnu-``

After this step the compiled binaries can be found in the sub-folder ``out``.


Compiler flags
**************
To be able to see all commands when building you could build using following
flags:

**GNU Make**

.. code-block:: bash

    $ make V=1

**CMake**

.. code-block:: bash

    $ make VERBOSE=1

Coding standards
****************
See :ref:`coding_standards`.

.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
