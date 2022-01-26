.. _arm_security_extensions:

#######################
Arm Security Extensions
#######################

.. _bti:

Branch Target Identification
****************************

Branch Target Identification (BTI) is an ARMv8.5 extension that provides
Control Flow Integrity (CFI) around indirect branches and their targets, thus helping
to limit the JOP (Jump Oriented Programming) attacks.

With this extension, ARM8.5-A introduces Branch Target Instructions (BTIs).
BTIs are also called landing pads. The processor can be configured so that
indirect branches (BR and BLR) only allows target landing pad instructions.
If the target of an indirect branch is not a landing pad, a Branch Target Exception 
is generated.

How to enable BTI for OP-TEE core
==================================

To make use of BTI in TEE core on CPU's that support it, enable the option 
``CFG_CORE_BTI``.

OP-TEE core makes use of some built-ins in the GCC/clang toolchains. So, in order
to use the option ``CFG_CORE_BTI``, make sure that GCC toolchain has been built with 
``--enable-standard-branch-protection`` is used else OP-TEE will fail to build.
By default libraries such as libgcc.a are built with flags (``-mbranch-protection=none``),
hence are incompatible with branch protection enabled. The Arm GNU compiler team
is looking for ways of providing users easy access to BTI-enabled libraries. 
In the short-term, they plan to create documentation to make it easier for users to
build BTI-enabled libraries themselves. Longer-term, they will begin discussions
on how to ensure BTI-enabled libraries are available automatically to users.
Please contact GCC team for more information on same. In the meantime, building a
BTI-enabled GCC toolchain is possible as decribed in :ref:`faq_gcc_bti`.

The same problem is also there with clang toolchain. So, when using clang to build
OP-TEE with ``CFG_CORE_BTI=y``, builtins (found in llvm's "compiler-rt"
project) must be built with BTI protection enabled. We have some instructions on
how to build the compiler-rt with BTI enabled. These are available in
:ref:`faq_llvm_bti`.


How to enable BTI for TA's
===========================

To make use of BTI support for TA's and user mode libraries, enable the option
``CFG_TA_BTI``. This will ensure that all libraries provided by OP-TEE to the TA's
as well as the TA's are built with BTI option.

When the TA's are loaded by ldelf, they are checked at run time for the BTI NOTE
property in ELF before enabling the protection for the TA.

When building TA's, you need to ensure that any external library used has been
built with branch-protection. This can be done by checking the library using readelf
command with option ``-n``. The BTI enabled libraries will have BTI NOTE property in
``.note.gnu.property`` section. If that is not the case, compilation will stop with a
warning. This is done intentionally to warn the user.


.. note::

        The BTI support is currently not compatible with options ``CFG_VIRTUALIZATION`` and
        ``CFG_WITH_PAGER``.
