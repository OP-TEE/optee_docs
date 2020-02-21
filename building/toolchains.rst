.. _toolchains:

##########
Toolchains
##########
OP-TEE uses both 32bit as well as 64bit toolchains and it is even possible to
mix them in some configurations. In theory you should be able to compile OP-TEE
with the Arm toolchains that are coming with your Linux distribution. But
instead of using those directly, we instead download the toolchains directly
from Arm.

Download/install
****************
We propose two ways to download the toolchains, both will put the toolchains
under the same path(s).

Direct download
===============
Go the `Arm GCC download page`_ and download the "`AArch32 target with soft
float (arm-linux-gnueabi)`" for 32bit builds and the "`AArch64 GNU/Linux target
(aarch64-linux-gnu)`" for 64bit builds. When the downloads have finished, you
will untar them to a location that you later on will export to your ``$PATH``.
Here is an example

.. code-block:: bash

    $ mkdir -p $HOME/toolchains
    $ cd $HOME/toolchains

    # Download 32bit toolchain
    $ wget https://developer.arm.com/-/media/Files/downloads/gnu-a/8.2-2019.01/gcc-arm-8.2-2019.01-x86_64-arm-linux-gnueabi.tar.xz
    $ mkdir aarch32
    $ tar xf gcc-arm-8.2-2019.01-x86_64-arm-linux-gnueabi.tar.xz -C aarch32 --strip-components=1

    # Download 64bit toolchain
    $ wget https://developer.arm.com/-/media/Files/downloads/gnu-a/8.2-2019.01/gcc-arm-8.2-2019.01-x86_64-aarch64-linux-gnu.tar.xz
    $ mkdir aarch64
    $ tar xf gcc-arm-8.2-2019.01-x86_64-aarch64-linux-gnu.tar.xz -C aarch64 --strip-components=1

Using build.git
===============
As an alternative, you can let :ref:`build`.git download them for you, but this
of course involves getting a git that you might not otherwise use.

.. code-block:: bash

    $ cd $HOME
    $ git clone https://github.com/OP-TEE/build.git
    $ cd build
    $ make -f toolchain.mk -j2


Export PATH
***********
If you have downloaded the toolchains as described above, you should have them
at ``$HOME/toolchains/{aarch32/aarch64}``, so now we just need to export the
paths and then you are ready to starting compiling OP-TEE components.

.. code-block:: bash

    $ export PATH=$PATH:$HOME/toolchains/aarch32/bin:$HOME/toolchains/aarch64/bin

.. _llvm:

LLVM / Clang
************
It's possible to also compile :ref:`optee_os`.git using llvm/clang. Either get
the toolchain using your package manager or alternatively build it yourself to
get the version that you need. To build the llvm toolchain for use in OP-TEE do
the following.

.. code-block:: bash

    $ sudo apt install ninja-build
    $ cd /tmp
    $ git clone https://github.com/llvm/llvm-project.git
    $ mkdir -p llvm-project/build
    $ cd llvm-project/build
    # Check out a version known to be working with OP-TEE
    $ git checkout llvmorg-9.0.1
    $ cmake -G Ninja \
        -DCMAKE_BUILD_TYPE=Release \
        -DLLVM_ENABLE_PROJECTS="clang;lld" \
        -DLLVM_TARGETS_TO_BUILD="AArch64;ARM" \
        -DCMAKE_INSTALL_PREFIX=<optee-project>/toolchains/clang-v9.0.1 ../llvm
    $ ninja
    $ ninja install

Now you'll have a llvm/clang toolchain ready to be used.

.. _Arm GCC download page: https://developer.arm.com/open-source/gnu-toolchain/gnu-a/downloads
