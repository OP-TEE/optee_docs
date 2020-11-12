.. _prerequisites:

#############
Prerequisites
#############
We believe that you can use any Linux distribution to build OP-TEE, but as
maintainers of OP-TEE we are mainly using Ubuntu-based distributions and to be
able to build and run OP-TEE there are a few packages that needs to be available.

First enable installation of i386 architecture packages and update the package
managers database.

.. code-block:: bash

    $ sudo dpkg --add-architecture i386
    $ sudo apt-get update

Install the following packages regardless of what target you will use in the end.

.. code-block:: bash

    $ sudo apt-get install android-tools-adb android-tools-fastboot autoconf \
            automake bc bison build-essential ccache cscope curl device-tree-compiler \
            expect flex ftp-upload gdisk iasl libattr1-dev libcap-dev \
            libfdt-dev libftdi-dev libglib2.0-dev libhidapi-dev libncurses5-dev \
            libpixman-1-dev libssl-dev libtool make \
            mtools netcat python-crypto python3-crypto python-pyelftools \
            python3-pycryptodome python3-pyelftools python-serial python3-serial \
            rsync unzip uuid-dev xdg-utils xterm xz-utils zlib1g-dev

For older versions, you might need to pull `pycryptodome` as a pip package:

.. code-block:: bash

    $ python3 -m pip install --user pycryptodome
