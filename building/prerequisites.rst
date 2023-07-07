.. _prerequisites:

#############
Prerequisites
#############
We believe that you can use any Linux distribution to build OP-TEE, but as
maintainers of OP-TEE we are mainly using Ubuntu-based distributions and to be
able to build and run OP-TEE there are a few packages that needs to be
available.

.. tabs::

    .. tab:: Ubuntu 22.04

        .. code-block:: bash

            $ sudo apt install \
              adb \
              acpica-tools \
              autoconf \
              automake \
              bc \
              bison \
              build-essential \
              ccache \
              cscope \
              curl \
              device-tree-compiler \
              e2tools \
              expect \
              fastboot \
              flex \
              ftp-upload \
              gdisk \
              libattr1-dev \
              libcap-dev \
              libfdt-dev \
              libftdi-dev \
              libglib2.0-dev \
              libgmp3-dev \
              libhidapi-dev \
              libmpc-dev \
              libncurses5-dev \
              libpixman-1-dev \
              libslirp-dev \
              libssl-dev \
              libtool \
              libusb-1.0-0-dev \
              make \
              mtools \
              netcat \
              ninja-build \
              python3-cryptography \
              python3-pip \
              python3-pyelftools \
              python3-serial \
              python-is-python3 \
              rsync \
              swig \
              unzip \
              uuid-dev \
              xdg-utils \
              xterm \
              xz-utils \
              zlib1g-dev


    .. tab:: Ubuntu 20.04

        .. code-block:: bash

            $ sudo apt install \
              android-tools-adb \
              android-tools-fastboot \
              autoconf \
              automake \
              bc \
              bison \
              build-essential \
              ccache \
              cpio \
              cscope \
              curl \
              device-tree-compiler \
              expect \
              flex \
              ftp-upload \
              gdisk \
              git \
              iasl \
              libattr1-dev \
              libcap-dev \
              libfdt-dev \
              libftdi-dev \
              libglib2.0-dev \
              libgmp3-dev \
              libhidapi-dev \
              libmpc-dev \
              libncurses5-dev \
              libpixman-1-dev \
              libslirp-dev \
              libssl-dev \
              libtool \
              make \
              mtools \
              netcat \
              ninja-build \
              python-is-python3 \
              python3-crypto \
              python3-cryptography \
              python3-pip \
              python3-pyelftools \
              python3-serial \
              rsync \
              unzip \
              uuid-dev \
              wget \
              xdg-utils \
              xterm \
              xz-utils \
              zlib1g-dev


    .. tab:: Older

        .. note::

            No longer supported by the OP-TEE community!

        Due to all changes over the years with different names of Python
        packages and different requirement in time for Python2 and/or Python3
        packages, it's not really possible to build more recent versions of
        OP-TEE with something that is older than Ubuntu 18.04. If you for some
        reason need to rebuild OP-TEE using a very old distro, then the best
        strategy for doing is is to check an earlier version of this
        documentation and start with the build instructions from there.
