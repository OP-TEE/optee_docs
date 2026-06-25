.. _aslr:

##########################################
Address Space Layout Randomization (ASLR)
##########################################

In :ref:`optee_os`, ASLR randomizes the runtime virtual address at which the
OP-TEE core is mapped and the user virtual address at which user mode
Trusted Applications (TAs) are loaded. That runtime address variation makes
many memory corruption bugs harder to exploit.

.. note::

   The options themselves are generic. This page also highlights the current
   RISC-V implementation where it differs from the generic code.

Pseudo TAs are part of the OP-TEE core image and therefore follow core ASLR,
not TA ASLR.

ASLR for OP-TEE core
********************

How to enable ASLR for OP-TEE core
==================================

Enable ``CFG_CORE_ASLR`` to randomize the runtime virtual address of the OP-TEE
core. The option defaults to ``y`` in ``mk/config.mk``, but platforms may force
another value. For example, ``core/arch/riscv/plat-spike/conf.mk`` forces
``CFG_CORE_ASLR=n``.

With ``CFG_CORE_ASLR=y``, the core is built as a position-independent
executable and ``tee.elf`` is linked with dynamic relocations enabled.

For deterministic testing, a fixed seed can be supplied with
``CFG_CORE_ASLR_SEED=<value>``. This option is restricted to insecure builds,
therefore ``CFG_INSECURE=y`` is required.

.. note::

   ``CFG_CORE_ASLR`` and ``CFG_CORE_SANITIZE_KADDRESS`` are not compatible.
   ``mk/config.mk`` rejects that combination directly.

RISC-V-specific notes
=====================

``-fpie`` and ``-pie`` are standard GCC and LLVM/Clang options. On RISC-V,
``CFG_CORE_ASLR=y`` maps to the following build flags:

.. list-table::
   :header-rows: 1

   * - Build stage
     - Argument
     - Effect
     - RISC-V build file
   * - Compile
     - ``-fpie``
     - Generate position-independent code suitable for executables
     - ``core/arch/riscv/riscv.mk``
   * - Link
     - ``-pie``
     - Link ``tee.elf`` as a position-independent executable with dynamic
       relocations enabled
     - ``core/arch/riscv/kernel/link.mk``

The primary hart selects the ASLR seed in
``core/arch/riscv/kernel/entry.S`` before the MMU is enabled:

- If ``CFG_CORE_ASLR_SEED`` is defined, that constant is used.
- Otherwise ``get_aslr_seed()`` is called.

The default ``get_aslr_seed()`` implementation in
``core/arch/riscv/kernel/boot.c`` first tries the RISC-V Zkr seed CSR path
when ``CFG_RISCV_ZKR_RNG=y`` and ``riscv_detect_csr_seed()`` reports support.
If that path is not available or fails, the code falls back to the weak
platform hook ``plat_get_aslr_seed()``.

The selected seed is passed to ``core_init_mmu_map()``. A zero seed disables
the randomized placement attempt. The generic MMU initialization code in
``core/mm/core_mmu.c`` then:

1. Builds the initial memory map for the core.
2. Tries up to three candidate virtual base addresses using
   ``arch_aslr_base_addr()`` from ``core/arch/riscv/mm/core_mmu_arch.c``.
3. Keeps the early identity-mapped initialization range so that the MMU can be
   enabled safely while execution still runs from identical physical and
   virtual addresses.

If randomized placement succeeds, ``boot_mmu_config.map_offset`` records the
offset from the link address. If no randomized placement works, or if the seed
is zero, the core falls back to its linked virtual address.

After the new mapping is prepared, ``entry.S`` performs the remaining RISC-V
specific steps:

1. ``relocate()`` applies the dynamic relocations while still executing from
   the identity-mapped region.
2. ``enable_mmu()`` enables ``satp`` and updates ``gp``, ``tp``, ``sp`` and
   ``ra`` with ``boot_mmu_config.map_offset`` so execution continues at the
   randomized virtual address.
3. ``boot_mem_relocate()`` fixes addresses stored in temporary boot
   allocations, and ``console_init()`` is run again because the registered
   console virtual address changed.

ASLR for user mode TAs
**********************

How to enable ASLR for TAs
===========================

Enable ``CFG_TA_ASLR`` to randomize the load address of user mode TAs handled
by ``ldelf``. This option defaults to ``y`` in ``mk/config.mk``.

The randomization range is controlled by:

- ``CFG_TA_ASLR_MIN_OFFSET_PAGES``
- ``CFG_TA_ASLR_MAX_OFFSET_PAGES``

The loader picks a page offset from ``CFG_TA_ASLR_MIN_OFFSET_PAGES`` up to,
but not including, ``CFG_TA_ASLR_MAX_OFFSET_PAGES``.
Larger ranges improve address diversity at the cost of more page table memory.

TA ASLR is implemented by the generic ELF loader in ``ldelf/ta_elf.c``.

When the first loadable segment of a TA or shared library is mapped,
``get_pad_begin()``:

1. Reads random bytes via ``sys_gen_random_num()``.
2. Converts the random value into a page count inside the configured ASLR
   range.
3. Returns the corresponding padding size in bytes.

Only the first mapping that establishes ``elf->load_addr`` receives the
random padding; subsequent segments are mapped relative to that load address.

That padding is passed to ``sys_map_ta_bin()``, ``sys_map_zi()`` or
``sys_remap()`` so the final ``elf->load_addr`` is shifted by a random page
offset.

If the mapping fails with ``TEE_ERROR_OUT_OF_MEMORY`` while using a non-zero
padding, ``ldelf`` retries the operation with zero padding. In that case ASLR
is effectively disabled for the current ELF file, but the TA can still be
loaded.

Once the load address is chosen, the rest of the TA load uses it
consistently:

- Segment virtual addresses are derived from ``elf->load_addr + seg->vaddr``.
- Relocations are resolved against ``elf->load_addr``.
- The final TA entry point returned by ``ta_elf_finalize_load_main()`` is
  ``elf->e_entry + elf->load_addr``.

The same logic applies to the main TA image and to additional ELF objects
loaded by ``ldelf``, subject to the same memory-availability fallback.

RISC-V-specific notes
=====================

RISC-V user mode TAs are built as position-independent code. The ELF layout is
defined in ``ta/arch/riscv/ta.ld.S``.

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

For ``-fpie`` and ``-pie`` semantics and toolchain support, see GCC_,
`GCC link options`_ and Clang_.

.. _GCC: https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html
.. _GCC link options: https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html
.. _Clang: https://clang.llvm.org/docs/ClangCommandLineReference.html
