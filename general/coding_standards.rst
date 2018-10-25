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

Regarding the checkpatch tool, it is not included directly into this project.
Please use checkpatch.pl from the Linux kernel git in combination with the local
`checkpatch script`_.

.. _checkpatch script: https://github.com/OP-TEE/optee_os/blob/master/scripts/checkpatch.sh
.. _checkpatch: http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/scripts/checkpatch.pl
.. _CodingStyle: https://www.kernel.org/doc/html/latest/process/coding-style.html
.. _repository-structure: fixme::after-sphinks-updates
