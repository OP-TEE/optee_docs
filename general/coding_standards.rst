.. _coding_standards:

Coding standards
################

In this project we are trying to adhere to the same coding convention as used
in the Linux kernel (see CodingStyle_). We achieve this by running
checkpatch_ from Linux kernel. However there are a few exceptions that we had
to make since the code also follows GlobalPlatform standards. The exceptions
are as follows:

    1. **CamelCase** for GlobalPlatform types is allowed.

    2. We **do not** run checkpatch on third party code that we might use in
       this project, such as LibTomCrypt, MPA, newlib etc. The reason for that
       and not doing checkpatch fixes for third party code is because we would
       probably deviate too much from upstream and therefore it would be hard to
       rebase against those projects later on and we don't expect that it is
       easy to convince other software projects to change coding style.

    3. **All** variables **shall be** initialized to a well known value in one
       or another way. The reason for that is that we have had potential
       security issues in the past that originated from not having variables
       initialized with a well defined value. We have also investigate various
       toolchain flags that are supposed to help out finding uninitialized
       variables. Unfortunately our conclusion is that you cannot trust the
       compilers here, since there are corner cases where compilers cannot
       reliably give a warning.

Variables are initialized according to these general guidelines:

    * Scalars (and types like ``time_t`` which are standardized as scalars)
      are initialized with ``0``, unless another value makes more sense.

    * For optee_client_ we need maximum portability. So use ``{ 0 }`` for
      struct types where the first element is known to be a scalar and
      ``memset()`` otherwise unless there is a good reason not to do so.

    * For the rest of the gits we assume that a recent version of GCC or
      Clang is used so we initialize structs with ``{ }`` in order to avoid
      the more clumsy ``memset()`` procedure. Types like ``pthread_t``
      which can be a scalar or a composite type are initialized with
      ``memset()`` in order to minimize the amount of future headache.
      Arrays are initialized with ``{ }``, too.

Unsigned integer constants are defined using the ``U()``, ``UL()`` or
``ULL()`` macros, depending on the required width. ``U()`` is a good choice
for 32-bit values.  Any of the minimum width cousins
``UINT{8,16,32,64}_C()`` are also accepted for compatibility. This makes
the sign and size of the integer well defined instead of depend on how
large the value is or which compiler is used. For example:

.. code-block:: c

    #define MY_UNSIGNED_CONSTANT U(123)

Running checkpatch
******************

Regarding the checkpatch tool, it is not included directly into this project.
Please use checkpatch.pl from the Linux kernel git in combination with the local
`checkpatch script`_. Environment variable CHECKPATCH is expected to provide
the path to the Linux checkpatch script path, i.e.:

.. code-block:: none

    export CHECKPATCH=/path/to/linux/scripts/checkpatch.pl
    ./scripts/checkpatch.sh HEAD
    ./scripts/checkpatch.sh --diff github/master HEAD

There are also targets for common use cases in the Makefiles:

.. code-block:: none

    export CHECKPATCH=/path/to/linux/scripts/checkpatch.pl
    make checkpatch         #check staging and working area
    make checkpatch-staging #check staging area (added, but not committed files)
    make checkpatch-working #check working area (modified, but not added files)

.. _checkpatch script: https://github.com/OP-TEE/optee_os/blob/master/scripts/checkpatch.sh
.. _checkpatch: http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/scripts/checkpatch.pl
.. _CodingStyle: https://www.kernel.org/doc/html/latest/process/coding-style.html
.. _optee_client: https://github.com/OP-TEE/optee_client
.. _repository-structure: fixme::after-sphinks-updates
