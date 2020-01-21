.. _quick_start_qemu:

#########################################
TL;DR --- Quick start instructions (QEMU)
#########################################
The following commands allow you to download, build and run OP-TEE in an
emulated 32-bit ARM environment (QEMU). They assume you are using a Ubuntu
18.04 Linux distribution (other distributions may need different packages).

Details and instructions for other platforms can be found on the main
:ref:`build_and_run` page.


.. literalinclude:: prerequisites.txt
   :language: bash

.. code-block:: bash

    $ mkdir optee
    $ cd optee
    $ repo init -u https://github.com/OP-TEE/manifest.git
    ...
    $ repo sync --no-clone-bundle
    ...
    $ cd build
    $ make toolchains
    ...
    $ make -j10 run
    ...
    (qemu) c
    ...

.. code-block:: bash

    Welcome to Buildroot, type root or test to login
    buildroot login: test
    $ xtest
    ...
