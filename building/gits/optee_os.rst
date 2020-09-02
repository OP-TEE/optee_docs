.. _optee_os:

########
optee_os
########

git location
************
https://github.com/OP-TEE/optee_os

License
*******
.. todo::

    Joakim: Necessary to state that here? Changing the "License headers" page to
    instead become a "License" page and add addtional sections.

The TEE core of optee_os is provided under the `BSD 2-Clause`_ license. But
there are also other software such as libraries included in optee_os. This
"other" software will have different licenses that are compatible with BSD
2-Clause (i.e., non-contaminating licenses unlike GPL-v2 for example).

.. _optee_os_build_system:

Build instructions
******************
You can build the code in this git only or build it as part of the entire
system, i.e. as a part of a full OP-TEE developer setup. For the latter, please
refer to instructions at the :ref:`build` page. For standalone builds optee_os
uses only regular GNU Makefiles (i.e. **no** CMake support here unlike the other
OP-TEE gits).

Configure the toolchain
=======================
First step is to download and configure a toolchain, see the :ref:`toolchains`
page for instructions.

Clone optee_os
==============
.. code-block:: bash

    $ git clone https://github.com/OP-TEE/optee_os
    $ cd optee_os

Build using GNU Make
====================
Since optee_os supports many devices and configurations it's impossible to give
a examples to all variants. But below is how you for example would build for
QEMU running Armv7-A (AArch32), with debugging enabled and the benchmark
framework disabled and will put all built files in a folder name `out/arm` in
the root of the git.

.. code-block:: bash
    :linenos:

    $ make \
        CFG_TEE_BENCHMARK=n \
        CFG_TEE_CORE_LOG_LEVEL=3 \
        CROSS_COMPILE=arm-linux-gnueabihf- \
        CROSS_COMPILE_core=arm-linux-gnueabihf- \
        CROSS_COMPILE_ta_arm32=arm-linux-gnueabihf- \
        CROSS_COMPILE_ta_arm64=aarch64-linux-gnu- \
        DEBUG=1 \
        O=out/arm \
        PLATFORM=vexpress-qemu_virt 

The same for an QEMU Armv8-A (AArch64) would look like this:

.. code-block:: bash
    :linenos:
    :emphasize-lines: 1,3,4,11

    $ make \
        CFG_ARM64_core=y \
        CFG_TEE_BENCHMARK=n \
        CFG_TEE_CORE_LOG_LEVEL=3 \
        CROSS_COMPILE=aarch64-linux-gnu- \
        CROSS_COMPILE_core=aarch64-linux-gnu- \
        CROSS_COMPILE_ta_arm32=arm-linux-gnueabihf- \
        CROSS_COMPILE_ta_arm64=aarch64-linux-gnu- \
        DEBUG=1 \
        O=out/arm \
        PLATFORM=vexpress-qemu_armv8a

.. hint::

    To be able to see all commands when building you could build with:

    .. code-block:: bash

        $ make V=1

Build using LLVM/clang
======================
optee_os can be compiled using llvm/clang. Start by downloading the toolchain
(see :ref:`llvm`). After that you can compile by running.

.. note::

    On line one you need to adjust the path so it matches the version of clang
    you are using.

.. code-block:: bash
    :linenos:
    :emphasize-lines: 1

    $ export PATH=<optee-project>/toolchains/clang-v9.0.1/bin:$PATH
    $ make COMPILER=clang


Coding standards
****************
See :ref:`coding_standards`.

Build system
************
The build system in optee_os consists of a main ``Makefile`` in the root of the
project together with ``sub.mk`` files in all source directories. In addition,
some supporting files are used to recursively process all ``sub.mk`` files and
generate the build rules.

+------------------------------------------------+-------------------------------+
| Name                                           | Description                   |
+================================================+===============================+
| ``core/core.mk``                               | Included from ``Makefile`` to |
|                                                | build the TEE Core            |
+------------------------------------------------+-------------------------------+
| ``ta/ta.mk``                                   | Included from ``Makefile`` to |
|                                                | create the TA devkit          |
+------------------------------------------------+-------------------------------+
| ``mk/compile.mk``                              | Create rules to make objects  |
|                                                | from source files             |
+------------------------------------------------+-------------------------------+
| ``mk/lib.mk``                                  | Create rules to make a        |
|                                                | libraries (.a)                |
+------------------------------------------------+-------------------------------+
| ``mk/subdir.mk``                               | Process ``sub.mk`` files      |
|                                                | recursively                   |
+------------------------------------------------+-------------------------------+
| ``mk/config.mk``                               | Global configuration variable |
+------------------------------------------------+-------------------------------+
| ``core/arch/$(ARCH)/$(ARCH).mk``               | Arch-specific compiler flags  |
+------------------------------------------------+-------------------------------+
| ``core/arch/$(ARCH)/plat-$(PLATFORM)/conf.mk`` | Platform-specific compiler    |
|                                                | flags and configuration       |
|                                                | variables                     |
+------------------------------------------------+-------------------------------+
| ``core/arch/$(ARCH)/plat-$(PLATFORM)/link.mk`` | Make recipes to link the TEE  |
|                                                | Core                          |
+------------------------------------------------+-------------------------------+
| ``ta/arch/arm/link.mk``                        | Make recipes to link Trusted  |
|                                                | Applications                  |
+------------------------------------------------+-------------------------------+
| ``ta/mk/ta_dev_kit.mk``                        | Main Makefile to be included  |
|                                                | when building Trusted         |
|                                                | Applications                  |
+------------------------------------------------+-------------------------------+
| ``mk/checkconf.mk``                            | Utility functions to          |
|                                                | manipulate configuration      |
|                                                | variables and generate        |
|                                                | a C header file               |
+------------------------------------------------+-------------------------------+
| ``sub.mk``                                     | List source files and define  |
|                                                | compiler flags                |
+------------------------------------------------+-------------------------------+

``make`` is always invoked from the top-level directory; there is no recursive
invocation of make itself.

Choosing the build target
=========================
The target architecture, platform and build directory may be selected by setting
environment or make variables (``VAR=value make`` or ``make VAR=value``).

ARCH - CPU architecture
-----------------------
``$(ARCH)`` is the CPU architecture to be built. Currently, the only supported
value is ``arm`` for 32-bit or 64-bit Armv7-A or Armv8-A. Please note that
contrary to the Linux kernel, ``$(ARCH)`` should **not** be set to ``arm64`` for
64-bit builds. The ``ARCH`` variable does not need to be set explicitly before
building either, because the proper instruction set is selected from the
``$(PLATFORM)`` value. For platforms that support both 32-bit and 64-bit builds,
``CFG_ARM64_core=y`` should be set to select 64-bit and not set (or set to
``n``) to select 32-bit.

Architecture-specific source code belongs to sub-directories that follow the
``arch/$(ARCH)`` pattern, such as: ``core/arch/arm``, ``lib/libutee/arch/arm``
and so on.

CROSS_COMPILE
-------------
``$(CROSS_COMPILE)`` is the prefix used to invoke the (32-bit) cross-compiler
toolchain. The default value is ``arm-linux-gnueabihf-``. This is the variable
you want to change in case you want to use ccache_ to speed you recompilations:

.. code-block:: bash

    $ make CROSS_COMPILE="ccache arm-linux-gnueabihf-"

If the build includes a mix of 32-bit and 64-bit code, for instance if you set
``CFG_ARM64_core=y`` to build a 64-bit secure kernel, then two different
toolchains are used, that are controlled by ``$(CROSS_COMPILE32)`` and
``$(CROSS_COMPILE64)``. The default value of ``$(CROSS_COMPILE32)`` is the value
of ``CROSS_COMPILE``, which defaults to ``arm-linux-gnueabihf-`` as mentioned
above. The default value of ``$(CROSS_COMPILE64)`` is ``aarch64-linux-gnu-``.
Examples:

.. code-block:: bash

    # For this example, select HiKey which supports both 32- and 64-bit builds
    $ export PLATFORM=hikey
    
    # 1. Build everything 32-bit
    $ make
    
    # 2. Same as (1.) but override the toolchain
    $ make CROSS_COMPILE="ccache arm-linux-gnueabihf-"
    
    # 3. Same as (2.)
    $ make CROSS_COMPILE32="ccache arm-linux-gnueabihf-"
    
    # 4. Select 64-bit secure 'core' (and therefore both 32- and 64-bit
    # Trusted Application libraries)
    $ make CFG_ARM64_core=y
    
    # 5. Same as (4.) but override the toolchains
    $ make CFG_ARM64_core=y \
           CROSS_COMPILE32="ccache arm-linux-gnueabihf-" \
           CROSS_COMPILE64="ccache aarch64-linux-gnu-"


.. _platform_flavor:

PLATFORM / PLATFORM_FLAVOR
--------------------------
A `platform` is a family of closely related hardware configurations. A platform
`flavor` is a variant of such configurations. When used together they define the
target hardware on which OP-TEE will be run.

For instance ``PLATFORM=stm PLATFORM_FLAVOR=b2260`` will build for the ST
Microelectronics 96boards/cannes2 board, while ``PLATFORM=vexpress
PLATFORM_FLAVOR=qemu_virt`` will generate code for a para-virtualized Arm
Versatile Express board running on QEMU.

For convenience, the flavor may be appended to the platform name with a dash, so
``make PLATFORM=stm-b2260`` is a shortcut for ``make PLATFORM=stm
PLATFORM_FLAVOR=b2260``. Note that in both cases the value of ``$(PLATFORM)`` is
``stm`` in the makefiles.

Platform-specific source code belongs to ``core/arch/$(ARCH)/plat-$(PLATFORM)``,
for instance: ``core/arch/arm/plat-vexpress`` or ``core/arch/arm/plat-stm``.

O - output directory
--------------------
All output files go into a platform-specific build directory, which is by
default ``out/$(ARCH)-plat-$(PLATFORM)``.

The output directory has basically the same structure as the source tree. For
instance, assuming ``ARCH=arm PLATFORM=stm``, ``core/kernel/panic.c`` will
compile into ``out/arm-plat-stm/core/kernel/panic.o``.

However, some libraries are compiled several times: once or twice for user mode,
and once for kernel mode. This is because they may be used by the TEE Core as
well as by the Trusted Applications. As a result, the ``lib`` source directory
gives two or three build directories: ``ta_arm{32,64}-lib`` and ``core-lib``.

The output directory also has an ``export-ta_arm{32,64}`` directory, which
contains:

    - All the files needed to build Trusted Applications.

        - In ``lib/``: ``libutee.a`` (the GlobalPlatform Internal API),
          ``libutils.a`` (which implements a part of the standard C library),
          and other libraries which provide additional APIs.

        - In ``include/``: header files for the above libraries

        - In ``mk/``: ``ta_dev_kit.mk``, which is a Make include file with
          suitable rules to build a TA, and its dependencies

        - ``scripts/sign.py``: a Python script used by ``ta_dev_kit.mk`` to sign
          TAs.

        - In ``src``: ``user_ta_header.c``: source file to add a suitable header
          to the Trusted Application (as expected by the loader code in the TEE
          Core).

    - Some files needed to build host applications (using the Client API), under
      ``export-ta_arm{32,64}/host_include``.

Finally, the build directory contains the auto-generated configuration file for
the TEE Core: ``$(O)/include/generated/conf.h`` (see below).

.. _configuration_and_flags:

Configuration and flags
=======================
The following variables are defined in ``core/arch/$(ARCH)/$(ARCH).mk``:

    - ``$(core-platform-aflags)``, ``$(core-platform-cflags)`` and
      ``$(core-platform-cppflags)`` are added to the assembler / C compiler /
      preprocessor flags for all source files compiled for TEE Core including
      the kernel versions of libraries such as ``libutils.a``.

    - ``$(ta_arm{32,64}-platform-aflags)``, ``$(ta_arm{32,64}-platform-cflags``)
      and ``$(ta_arm{32,64}-platform-cppflags)`` are added to the assembler / C
      compiler / preprocessor flags when building the user-mode libraries
      (``libutee.a``, ``libutils.a``) or Trusted Applications.

      The following variables are defined in
      ``core/arch/$(ARCH)/plat-$(PLATFORM)/conf.mk``:

    - If ``$(arm{32,64}-platform-cflags)``, ``$(arm{32,64}-platform-aflags)``
      and ``$(arm{32,64}-platform-cppflags)`` are defined their content will be
      added to ``$(\*-platform-\*flags)`` when they are are initialized in
      ``core/arch/$(ARCH)/$(ARCH).mk`` as described above.

    - ``$(core-platform-subdirs)`` is the list of the subdirectories that are
      added to the TEE Core.

Linker scripts
==============
The file ``core/arch/$(ARCH)/plat-$(PLATFORM)/link.mk`` contains the rules to
link the TEE Core and perform any related tasks, such as running ``objdump`` to
produce a dump file. ``link.mk`` adds files to the ``all:`` target.

Source files
============
Each directory that contains source files has a file called ``sub.mk``. This
makefile defines the source files that should be included in the build, as well
as any subdirectories that should be processed, too. For example:

.. code-block:: make

    # core/arch/arm/sm/sub.mk
    srcs-y += sm_asm.S
    srcs-y += sm.c

.. code-block:: make

    # core/sub.mk
    subdirs-y += kernel
    subdirs-y += mm
    subdirs-y += tee
    subdirs-y += drivers

The ``-y`` suffix is meant to facilitate conditional compilation. See
section :ref:`configuration_variables` below.

``srcs-y`` and ``subdirs-y`` are often not used together in the same ``sub.mk``,
because source files are usually alone in leaf directories. But this is not a
hard rule.

In addition to source files, ``sub.mk`` may define compiler flags, include
directories and/or configuration variables as explained below.

Compiler flags
==============
Default compiler flags are defined in ``mk/compile.mk``. Note that
platform-specific flags must not appear in this file which is common to all
platforms.

To add flags for a given source file, you may use the following variables in
``sub.mk``:

    - ``cflags-<filename>-y`` for C files (\*.c)

    - ``aflags-<filename>-y`` for assembler files (\*.S)

    - ``cppflags-<filename>-y`` for both C and assembler

For instance:

.. code-block:: make

    # core/lib/libtomcrypt/src/pk/dh/sub.mk
    srcs-y += dh.c
    cflags-dh.c-y := -Wno-unused-variable

Compiler flags may also be removed, as follows:

.. code-block:: make

    # lib/libutils/isoc/newlib/sub.mk
    srcs-y += memmove.c
    cflags-remove-memmove.c-y += -Wcast-align

Some variables apply to libraries only (that is, when using ``mk/lib.mk``) and
affect all the source files that belong to the library: ``cppflags-lib-y`` and
``cflags-lib-y``.

Include directories
===================
Include directories may be added to ``global-incdirs-y``, in which case they
will be accessible from all the source files and will be copied to
``export-ta_arm{32,64}/include`` and ``export-ta_arm{32,64}/host_include``.

When ``sub.mk`` is used to build a library, ``incdirs-lib-y`` may receive
additional directories that will be used for that library only.

.. _configuration_variables:

Configuration variables
=======================
Some features may be enabled, disabled or otherwise controlled at compile time
through makefile variables. Default values are normally provided in makefiles
with the ``?=`` operator so that their value may be easily overridden by
environment variables. For instance:

.. code-block:: make

    PLATFORM ?= stm
    PLATFORM_FLAVOR ?= default

Some global configuration variables are defined in ``mk/config.mk``, but others
may be defined in ``sub.mk`` when then pertain to a specific library for
instance.

Variables with the ``CFG_`` prefix are treated in a special way: their value is
automatically reflected in the generated header file
``$(out-dir)/include/generated/conf.h``, after all the included makefiles have
been processed. ``conf.h`` is automatically included by the preprocessor when a
source file is built.

Depending on their value, variables may be considered either boolean or
non-boolean, which affects how they are translated into ``conf.h``.

Boolean configuration variables
-------------------------------
When a configuration variable controls the presence or absence of a feature,
``y`` means **enabled**, while ``n``, empty value or an undefined variable means
**disabled**. For instance, the following commands are equivalent and would
disable feature ``CFG_CRYPTO_GCM``:

.. code-block:: bash

    $ make CFG_CRYPTO_GCM=n

.. code-block:: bash

    $ make CFG_CRYPTO_GCM=

.. code-block:: bash

    $ CFG_CRYPTO_GCM=n make

.. code-block:: bash

    $ export CFG_CRYPTO_GCM=n
    $ make

Configuration variables may then be used directly in ``sub.mk`` to trigger
conditional compilation:

.. code-block:: make

    # core/lib/libtomcrypt/src/encauth/sub.mk
    subdirs-$(CFG_CRYPTO_CCM) += ccm
    subdirs-$(CFG_CRYPTO_GCM) += gcm

When a configuration variable is **enabled** (``y``), ``<generated/conf.h>``
contains a macro with the same name as the variable and the value ``1``. If it
is  **disabled**, however, no macro definition is output. This allows the C code
to use constructs like:

.. code-block:: c

    /* core/lib/libtomcrypt/src/tee_ltc_provider.c */

    /* ... */

    #if defined(CFG_CRYPTO_GCM)
    struct tee_gcm_state {
            gcm_state ctx;      /* the gcm state as defined by LTC */
            size_t tag_len;     /* tag length */
    };
    #endif

Non-boolean configuration variables
-----------------------------------
Configuration variables that are not recognized as booleans are simply output
unchanged into `<generated/conf.h>`. For instance:

.. code-block:: bash

    $ make CFG_TEE_CORE_LOG_LEVEL=4

.. code-block:: c

    /* out/arm-plat-vexpress/include/generated/conf.h */

    #define CFG_TEE_CORE_LOG_LEVEL 4 /* '4' */

The 'force' macro
-----------------

Some platforms may require specific values for some of their configuration
variables. For instance, the number of CPU cores in a system is fixed so the
value of ``CFG_TEE_CORE_NB_CORE`` should generally not be changed. Or some
feature may not be supported by the hardware, in which case the corresponding
configuration variable should always be disabled.

In such cases, the ``force`` macro should be used. It sets the variable to the
specified value while reporting about any conflicting value that may have been
set previously, either via a previous assignment in the makefiles or via the
command line or environment variables. For example:

.. code-block:: make

    $(call force,CFG_TEE_CORE_NB_CORE,8)
    $(call force,CFG_ARM64_core,n)
    $(call force,CFG_ARM_GICV3,y)

.. code-block:: bash

    $ make -j10 PLATFORM=hikey CFG_TEE_CORE_NB_CORE=4
    core/arch/arm/plat-hikey/conf.mk:5: *** CFG_TEE_CORE_NB_CORE is set to '4' (from command line) but its value must be '8'.  Stop.

Configuration dependencies
--------------------------
Some combinations of configuration variables may not be valid. This should be
dealt with by custom checks in makefiles. ``mk/checkconf.h`` provides functions
to help detect and deal with such situations.

Import branches
***************
This section is more specifically intended for maintainers.

The optee_os repository contains branches with the ``import/`` prefix,
which we call *import branches* below. This section describes their purpose and
how they are used.

Import branches are meant to help import external libraries into the
optee_os repository and maintain them:

 - *Import* means copy source files from a given upstream version of the
   library and commit them locally (typically under ``optee_os/lib`` or
   ``optee_os/core/lib``), along with OP-TEE specific changes (build and
   configuration files for instance)
 - *Maintain* means carry local bug fixes or improvements that did not make
   their way upstream, and periodically upgrade the library by importing
   changes from upstream.

Import branches have the version of the imported library in their names. For
example: ``import/mbedtls-2.6.1``. They are forked from ``master``. They record
the history of all changes made for OP-TEE to a library for a given library
version. For example, the import branch for Mbed TLS 2.6.1 illustrates how the
Mbed TLS library was initially imported and later modified:

.. code-block:: none

    $ BRANCH=github/import/mbedtls-2.6.1
    $ BASE=`git merge-base master $BRANCH`
    $ git log --oneline --no-merges $BASE..$BRANCH
    8ff963a6 (github/import/mbedtls-2.6.1) mbedtls: fix memory leak in mpi_miller_rabin()
    213cce52 libmedtls: mpi_miller_rabin: increase count limit
    f934e291 mbedtls: add mbedtls_mpi_init_static()
    782fddd1 libmbedtls: add mbedtls_mpi_init_mempool()
    33873834 libmbedtls: make mbedtls_mpi_mont*() available
    e0186224 libmbedtls: refine mbedtls license header
    215609ae mbedtls: configure mbedtls to reach for config
    6916dcd9 mbedtls: remove default include/mbedtls/config.h
    b60fc42a Import mbedtls-2.6.1

Commit b60fc42a imports the library under ``lib/libmbedtls/mbedtls`` with no
modification to the code (not the whole library is imported however, since some
files are not needed they are deleted as mentioned in the commit description).
Then a couple of adjustments are made in commits 6916dcd9 and 215609ae in order
to be able to build with optee_os. At this point the initial import is done and
subsequent commits are local improvements or bug fixes made later on.

The initial import (commits b60fc42a, 6916dcd9 and 215609ae) is merged into
master as a "squashed" commit to preserve bisectability -- in other words, so
that no commit in master breaks the build:

.. code-block:: none

    $ git show --quiet 817466cb
    commit 817466cb476de705a8e3dabe1ef165fe27a18c2f
    Author: Jens Wiklander <jens.wiklander@linaro.org>
    Date:   Tue May 22 13:49:31 2018 +0200

        Squashed commit importing mbedtls-2.6.1 source
    
        Squash merging branch import/mbedtls-2.6.1
    
        215609ae4d8c ("mbedtls: configure mbedtls to reach for config")
        6916dcd9b9cd ("mbedtls: remove default include/mbedtls/config.h")
        b60fc42a5cd5 ("Import mbedtls-2.6.1")
    
        Acked-by: Joakim Bech <joakim.bech@linaro.org>
        Signed-off-by: Jens Wiklander <jens.wiklander@linaro.org>

Later changes are reviewed and merged into master normally, and are also
recorded on top of the import branch via pull requests against the import
branch. Consider for example the two following commits, one is in the master
branch and the other is applied on top of the import branch.

.. code-block:: none

    18c5148d357e ("mbedtls: add mbedtls_mpi_init_static()")  # In master
    f934e2913b7b ("mbedtls: add mbedtls_mpi_init_static()")  # In import/mbedtls-2.6.1

The master branch is occasionally merged into the import branches. Otherwise,
some patches cherry-picked from master would not apply. Note that it is a
"normal" merge (not a rebase), so the commits that are on master can easily
be filtered out (``git log --oneline --first-parent import/...``).

When it is time to upgrade a library, a new import branch is created from
master, for example: ``import/mbedtls-2.16.0``. A pull request is created against
this branch with the following commits:

 - The first commit deletes the "old" version of the library and imports the
   new upstream version.
 - The subsequent commits are cherry-picked from the previous import branch and
   adjusted as needed. These commits are effectively "rebased" onto the new
   library.
 - Build files are updated if needed.

Here is the history of the ``import/mbedtls-2.16.0`` branch, for comparison with
the initial import:

.. code-block:: none

    $ BRANCH=github/import/mbedtls-2.16.0
    $ BASE=`git merge-base master $BRANCH`
    $ git log --oneline --no-merges $BASE..$BRANCH
    68df6eb0 libmbedtls: mbedtls_mpi_exp_mod(): reduce stack usage
    f58facc6 libutee: increase MPI mempool size
    be040a3e libmbedtls: preserve mempool usage on reinit
    ae499f6a libmbedtls: mbedtls_mpi_exp_mod() initialize W
    b95a6c5d libmbedtls: fix no CRT issue
    ac34734a libmbedtls: add interfaces in mbedtls for context memory operation
    9ee2a92d libmbedtls: compile new files added with 2.16.0
    9b0818d4 mbedtls: fix memory leak in mpi_miller_rabin()
    2d6644ee libmedtls: mpi_miller_rabin: increase count limit
    d831db4c libmbedtls: add mbedtls_mpi_init_mempool()
    df0f4886 libmbedtls: make mbedtls_mpi_mont*() available
    7b079206 libmbedtls: refine mbedtls license header
    2616e2d9 mbedtls: configure mbedtls to reach for config
    d686ab1c mbedtls: remove default include/mbedtls/config.h
    50a57cfa Import mbedtls-2.16.0
    8bfc3de4 libutee: lessen dependency on mbedtls internals

Note that the first commit 8bfc3de4 ("libutee: lessen dependency on mbedtls
internals") can be ignored, it was applied to master in anticipation of the
2.16.0 upgrade but the ``import/mbedtls-2.16.0`` was forked before.

The upgrade from 2.6.1 to 2.16.0 is made of all the commits up to and
including commit 9ee2a92d ("libmbedtls: compile new files added with 2.16.0"):

 - Commit 50a57cfa ("Import mbedtls-2.16.0") deletes the "old" files and
   library imports the new ones.
 - Commits d686ab1c..9b0818d4 are cherry-picked from the previous import
   branch.
 - Commit 9ee2a92d ("libmbedtls: compile new files added with 2.16.0") adapts
   the build files to the new version.

The master branch contains a squashed equivalent of the above:

.. code-block:: none

    $ git show --quiet 3d3b0591
    commit 3d3b05918ec9052ba13de82fbcaba204766eb636
    Author: Jens Wiklander <jens.wiklander@linaro.org>
    Date:   Wed Mar 20 15:30:29 2019 +0100

        Squashed commit upgrading to mbedtls-2.16.0
       
        Squash merging branch import/mbedtls-2.16.0
       
        9ee2a92de51f ("libmbedtls: compile new files added with 2.16.0")
        9b0818d48d29 ("mbedtls: fix memory leak in mpi_miller_rabin()")
        2d6644ee0bbe ("libmedtls: mpi_miller_rabin: increase count limit")
        d831db4c238a ("libmbedtls: add mbedtls_mpi_init_mempool()")
        df0f4886b663 ("libmbedtls: make mbedtls_mpi_mont*() available")
        7b0792062b65 ("libmbedtls: refine mbedtls license header")
        2616e2d9709f ("mbedtls: configure mbedtls to reach for config")
        d686ab1c51b7 ("mbedtls: remove default include/mbedtls/config.h")
        50a57cfac892 ("Import mbedtls-2.16.0")
   
        Acked-by: Jerome Forissier <jerome.forissier@linaro.org>
        Signed-off-by: Jens Wiklander <jens.wiklander@linaro.org>

Subsequent commits in the ``import/mbedtls-2.16.0`` branch are modifications
that happened later in master as a result of OP-TEE development.

.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
.. _ccache: https://ccache.samba.org/
