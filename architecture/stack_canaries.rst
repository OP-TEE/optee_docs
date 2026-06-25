.. _stack_canaries:

####################################
Compiler-instrumented Stack Canaries
####################################

With stack protector enabled, the compiler saves a guard value in selected
stack frames and checks it again before returning. If the value has changed,
the function calls ``__stack_chk_fail()``. In OP-TEE, the reference guard is
stored in ``__stack_chk_guard``.

The compiler arguments below are standard stack protector options supported by
both GCC and LLVM/Clang.

.. list-table::
   :header-rows: 1

   * - Compiler argument
     - Effect
   * - ``-fstack-protector``
     - Protects functions selected by the compiler's basic stack protector
       heuristics.
   * - ``-fstack-protector-strong``
     - Extends protection to more functions, including ones with local arrays,
       ``alloca()`` calls or address-taken local variables.
   * - ``-fstack-protector-all``
     - Protects all functions.
   * - ``-fno-stack-protector``
     - Disables stack protector instrumentation.

.. note::

   This is separate from the thread stack boundary canaries in
   ``core/kernel/thread.c`` controlled by ``CFG_WITH_STACK_CANARIES``.

Pseudo TAs are part of the OP-TEE core image and therefore follow core stack
protector settings, not TA stack protector settings.

Stack canaries for OP-TEE core
******************************

How to enable stack canaries for OP-TEE core
============================================

The OP-TEE core build accepts the following options:

The ``CFG_*`` option enables the matching compiler argument. The ``Build
logic`` column shows where that argument is added to the build. These options
are defined in ``mk/config.mk``.

These flags are added by the common core build logic in ``core/core.mk``. They
are not specific to RISC-V; Arm uses the same selection mechanism.

.. list-table::
   :header-rows: 1

   * - CFG name
     - Compiler argument
     - Build logic
     - Default
   * - ``CFG_CORE_STACK_PROTECTOR``
     - ``-fstack-protector``
     - ``core/core.mk``
     - ``n``
   * - ``CFG_CORE_STACK_PROTECTOR_STRONG``
     - ``-fstack-protector-strong``
     - ``core/core.mk``
     - ``y``
   * - ``CFG_CORE_STACK_PROTECTOR_ALL``
     - ``-fstack-protector-all``
     - ``core/core.mk``
     - ``n``

With none of the three options selected, the core build uses
``-fno-stack-protector``.

Core implementation on RISC-V
=============================

RISC-V platforms use the default weak implementation of
``plat_get_random_stack_canaries()`` from ``core/kernel/boot.c``. It fills one
or more canaries with ``crypto_rng_read()`` and then clears the low byte of
each value. This leaves a null byte in the guard, which helps against some
string-based overwrite patterns.

With ``CFG_NS_VIRTUALIZATION=y``, the default weak implementation cannot use
the core RNG at that point and falls back to a fixed value, while printing a
warning. Platforms using virtualization should override
``plat_get_random_stack_canaries()`` if compiler stack protection or thread
stack boundary canaries are required.

After ``boot_init_primary_final()``, the RISC-V entry code in
``core/arch/riscv/kernel/entry.S`` fetches one native-word canary and writes
it to ``__stack_chk_guard``. The guard variable and ``__stack_chk_fail()``
live in ``lib/libutils/isoc/stack_check.c``. A failed check prints
``stack smashing detected`` and then panics the core.

Stack canaries for user mode TAs
********************************

How to enable stack canaries for TAs
====================================

User mode TAs and the TA dev kit libraries accept the following options:

The ``CFG_*`` option enables the matching compiler argument. The ``Build
logic`` column shows where that argument is added to the build. These options
are defined in ``mk/config.mk``.

These flags are added by the common TA build logic in ``ta/ta.mk``. They are
not specific to RISC-V, and the same mechanism is used for other supported TA
architectures.

.. list-table::
   :header-rows: 1

   * - CFG name
     - Compiler argument
     - Build logic
     - Default
   * - ``CFG_TA_STACK_PROTECTOR``
     - ``-fstack-protector``
     - ``ta/ta.mk``
     - ``n``
   * - ``CFG_TA_STACK_PROTECTOR_STRONG``
     - ``-fstack-protector-strong``
     - ``ta/ta.mk``
     - ``y``
   * - ``CFG_TA_STACK_PROTECTOR_ALL``
     - ``-fstack-protector-all``
     - ``ta/ta.mk``
     - ``n``

With none of the three options selected, the TA build uses
``-fno-stack-protector``.

The TA startup code in ``ta/user_ta_header.c`` initializes the guard lazily in
``__ta_entry()`` before dispatching to ``__utee_entry()``.

``__ta_entry()`` is marked ``__no_stack_protector`` so it can seed the guard
before any compiler-generated canary check applies to that function itself.

When ``_CFG_TA_STACK_PROTECTOR`` is enabled and the TA enters for the first
time, ``__ta_entry()`` calls ``_utee_cryp_random_number_generate()`` for a
``uintptr_t`` canary, clears its low byte to leave a null byte in the guard,
stores the result into ``__stack_chk_guard``, and records that the guard has
been initialized.

Subsequent entries reuse the same guard for the lifetime of the TA instance.

If a protected TA function detects corruption, ``__stack_chk_fail()`` in
``lib/libutils/isoc/stack_check.c`` prints ``stack smashing detected`` and
calls ``_utee_panic(TEE_ERROR_OVERFLOW)``.

TA implementation on RISC-V
===========================

RISC-V user mode TAs use the generic TA stack protector initialization path in
``ta/user_ta_header.c``.

User mode TA support is still platform-specific on RISC-V:

.. list-table::
   :header-rows: 1

   * - Platform file
     - User mode TA support
   * - ``core/arch/riscv/plat-virt/conf.mk``
     - Exposes ``ta_rv64``
   * - ``core/arch/riscv/plat-sifive/conf.mk``
     - Exposes ``ta_rv64``
   * - ``core/arch/riscv/plat-spike/conf.mk``
     - Forces ``CFG_WITH_USER_TA=n``

References
**********

For compiler-side flag semantics, see GCC_ and Clang_.

.. _GCC: https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html
.. _Clang: https://clang.llvm.org/docs/ClangCommandLineReference.html
