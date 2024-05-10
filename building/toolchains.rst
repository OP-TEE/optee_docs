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
Go the `Arm GNU Toolchain Downloads page`_ and download the "`AArch32 target with soft
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
It's possible to also compile :ref:`optee_os`.git using llvm/clang. To do that,
you can download and extract Clang from the GitHub release page. You'll need an
x86_64 cross-compiler capable of generating aarch64 and armv7a code **and** the
compiler-rt libraries for these architectures (libclang_rt.*.a).

Clang is configured to be able to cross-compile to all the supported
architectures by default (see <clang path>/bin/llc --version) which is great,
but compiler-rt is included only for the host architecture. Therefore you need
to combine several packages into one. Please refer to this `get_clang.sh`_
script for details on creating a llvm/clang toolchain ready to be used.

Using build.git
===============
As an alternative, you can let :ref:`build`.git download them for you, but this
of course involves getting a git that you might not otherwise use.

.. code-block:: bash

    $ cd $HOME
    $ git clone https://github.com/OP-TEE/build.git
    $ cd build
    $ make -f toolchain.mk clang-toolchains

The above instructions will download and install Clang in ``$HOME/clang-9.0.1``.

You can also get the toolchain using your package manager or alternatively build
it yourself, but these alternative methods risk being incomplete. For example,
the Ubuntu clang package does not install the needed ld.lld package. The package
also does not contain the cross-compiled compiler-rt libraries. Building by
yourself is hard for the same reason, i.e. no cross-compiled compiler-rt
libraries are generated.


.. _Arm GNU Toolchain Downloads page: https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads
.. _get_clang.sh: https://github.com/OP-TEE/build/blob/master/get_clang.sh
