.. _prerequisites:

#############
Prerequisites
#############
We believe that you can use any Linux distribution to build OP-TEE, but as
maintainers of OP-TEE we are mainly using Ubuntu-based distributions and to be
able to build and run OP-TEE there are a few packages that needs to be installed
to start with. Therefore install the following packages regardless of what
target you will use in the end.

.. code-block:: bash

    $ sudo apt-get install android-tools-adb android-tools-fastboot autoconf \
            automake bc bison build-essential cscope curl device-tree-compiler \
            expect flex ftp-upload gdisk iasl libattr1-dev libc6:i386 libcap-dev \
            libfdt-dev libftdi-dev libglib2.0-dev libhidapi-dev libncurses5-dev \
            libpixman-1-dev libssl-dev libstdc++6:i386 libtool libz1:i386 make \
            mtools netcat python-crypto python-serial python-wand unzip uuid-dev \
            xdg-utils xterm xz-utils zlib1g-dev
