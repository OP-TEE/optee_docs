.. _core:

####
Core
####

.. _interrupt_handling:

Interrupt handling
******************
This section describes how :ref:`optee_os` handles switches of world execution
context based on :ref:`SMC` exceptions and interrupt notifications Interrupt
notifications are IRQ/FIQ exceptions which may also imply switching of world
execution context: normal world to secure world, or secure world to normal
world.

Use cases of world context switch
=================================
This section lists all the cases where optee_os is involved in world context
switches. Optee_os executes in the secure world. World switch is done by the
cores secure monitor level/mode, referred below as the Monitor.

When the normal world invokes the secure world, the normal world executes a SMC
instruction. The SMC exception is always trapped by the Monitor. If the related
service targets the trusted OS, the Monitor will switch to optee_os world
execution. When the secure world returns to the normal world, optee_os executes
a SMC that is caught by the Monitor which switches back to the normal world.

When a secure interrupt is signaled by the Arm GIC, it shall reach the optee_os
interrupt exception vector. If the secure world is executing, optee_os will
handle straight the interrupt from its exception vector. If the normal world is
executing when the secure interrupt raises, the Monitor vector must handle the
exception and invoke the optee_os to serve the interrupt.

When a non-secure interrupt is signaled by the Arm GIC, it shall reach the
normal world interrupt exception vector. If the normal world is executing, it
will handle straight the exception from its exception vector. If the secure
world is executing when the non-secure interrupt raises, optee_os will
temporarily return back to normal world via the Monitor to let normal world
serve the interrupt.

Core exception vectors
======================
Monitor vector is ``VBAR_EL3`` in AArch64 and ``MVBAR`` in Armv7-A/AArch32.
Monitor can be reached while normal world or secure world is executing. The
executing secure state is known to the Monitor through the ``SCR_NS``.

Monitor can be reached from a SMC exception, an IRQ or FIQ exception (so-called
interrupts) and from asynchronous aborts. Obviously monitor aborts (data,
prefetch, undef) are local to the Monitor execution.

The Monitor can be external to optee_os (case ``CFG_WITH_ARM_TRUSTED_FW=y``).
If not, provides a local secure monitor ``core/arch/arm/sm``. Armv7-A platforms
should use the optee_os secure monitor. Armv8-A platforms are likely to rely on
an `Trusted Firmware A`_.

When executing outside the Monitor, the system is executing either in the
normal world (``SCR_NS=1``) or in the secure world (``SCR_NS=0``). Each world
owns its own exception vector table (state vector):

    - ``VBAR_EL2`` or ``VBAR_EL1`` non-secure or ``VBAR_EL1`` secure for
      AArch64.
    - ``HVBAR`` or ``VBAR`` non-secure or ``VBAR`` secure for Armv7-A and
      AArch32.

All SMC exceptions are trapped in the Monitor vector. IRQ/FIQ exceptions can be
trapped either in the Monitor vector or in the state vector of the executing
world.

When the normal world is executing, the system is configured to route:

    - secure interrupts to the Monitor that will forward to optee_os
    - non-secure interrupts to the executing world exception vector.

When the secure world is executing, the system is configured to route:

    - secure and non-secure interrupts to the executing optee_os exception
      vector. optee_os shall forward the non-secure interrupts to the normal
      world.

Optee_os non-secure interrupts are always trapped in the state vector of the
executing world. This is reflected by a static value of ``SCR_(IRQ|FIQ)``.

.. _native_foreign_irqs:

Native and foreign interrupts
=============================
Two types of interrupt are defined in optee_os:

    - **Native interrupt** - The interrupt handled by optee_os (for example:
      secure interrupt)
    - **Foreign interrupt** - The interrupt not handled by optee_os (for
      example: non-secure interrupt which is handled by normal world)

For Arm **GICv2** mode, native interrupt is sent as FIQ and foreign interrupt
is sent as IRQ. For Arm **GICv3** mode, foreign interrupt is sent as FIQ which
could be handled by either secure world (aarch32 Monitor mode or aarch64 EL3)
or normal world. Arm GICv3 mode can be enabled by setting ``CFG_ARM_GICV3=y``.
For clarity, this document mainly chooses the GICv2 convention and refers the
IRQ as optee_os foreign interrupts, and FIQ as optee_os native interrupts.
Native interrupts must be securely routed to optee_os. Foreign interrupts, when
trapped during secure world execution might need to be efficiently routed to
the normal world.

Normal World invokes optee_os using SMC
=======================================

**Entering the Secure Monitor**

The monitor manages all entries and exits of secure world. To enter secure
world from normal world the monitor saves the state of normal world (general
purpose registers and system registers which are not banked) and restores the
previous state of secure world. Then a return from exception is performed and
the restored secure state is resumed. Exit from secure world to normal world is
the reverse.

Some general purpose registers are not saved and restored on entry and exit,
those are used to pass parameters between secure and normal world (see
ARM_DEN0028A_SMC_Calling_Convention_ for details).

**Entry and exit of Trusted OS**

On entry and exit of Trusted OS each CPU is uses a separate entry stack and runs
with IRQ and FIQ blocked. SMCs are categorised in two flavors: **fast** and
**standard**.

    - For **fast** SMCs, optee_os will execute on the entry stack with IRQ/FIQ
      blocked until the execution returns to normal world.

    - For **standard** SMCs, optee_os will at some point execute the requested
      service with interrupts unblocked. In order to handle interrupts, mainly
      forwarding of foreign interrupts, optee_os assigns a trusted thread
      (`core/arch/arm/kernel/thread.c`_) to the SMC request. The trusted thread
      stores the execution context of the requested service. This context can be
      suspended and resumed as the requested service executes and is
      interrupted. The trusted thread is released only once the service
      execution returns with a completion status.

      For **standard** SMCs, optee_os allocates or resumes a trusted thread then
      unblock the IRQ/FIQ lines. When the optee_os needs to invoke the normal
      world from a foreign interrupt or a remote service call, optee_os blocks
      IRQ/FIQ and suspends the trusted thread. When suspending, optee_os gets
      back to the entry stack.

    - **Both** fast and standard SMC end on the entry stack with IRQ/FIQ blocked
      and optee_os invokes the Monitor through a SMC to return to the normal
      world.

.. figure:: ../images/core/interrupt_handling/tee_invoke.png
    :figclass: align-center
    
    SMC entry to secure world

Deliver non-secure interrupts to Normal World
=============================================
This section uses the Arm GICv1/v2 conventions: IRQ signals non-secure
interrupts while FIQ signals secure interrupts. On a GICv3 configuration, one
should exchange IRQ and FIQ in this section.

**Forward a Foreign Interrupt from Secure World to Normal World**

When an IRQ is received in secure world as an IRQ exception then secure world:

    1. Saves trusted thread context (entire state of all processor modes for
       Armv7-A)

    2. Blocks (masks) all interrupts (IRQ and FIQ)

    3. Switches to entry stack

    4. Issues an SMC with a value to indicates to normal world that an IRQ has
       been delivered and last SMC call should be continued

The monitor restores normal world context with a return code indicating that an
IRQ is about to be delivered. Normal world issues a new SMC indicating that it
should continue last SMC.

The monitor restores secure world context which locates the previously saved
context and checks that it is a return from IRQ that is requested before
restoring the context and lets the secure world IRQ handler return from
exception where the execution would be resumed.

Note that the monitor itself does not know/care that it has just forwarded an
IRQ to normal world. The bookkeeping is done in the trusted thread handling in
Trusted OS. Normal world is responsible to decide when the secure world thread
should resume execution (for details, see :ref:`thread_handling`).

.. figure:: ../images/core/interrupt_handling/irq.png
    :figclass: align-center
    
    IRQ received in secure world and forwarded to normal world

**Deliver a non-secure interrupt to normal world when ``SCR_NS`` is set**

Since ``SCR_IRQ`` is cleared, an IRQ will be delivered using the state vector
(``VBAR``) in the normal world. The IRQ is received as any other exception by
normal world, the monitor and the Trusted OS are not involved at all.

Deliver secure interrupts to Secure World
=========================================
This section uses the Arm GICv1/v2 conventions: FIQ signals secure interrupts
while IRQ signals non-secure interrupts. On a GICv3 configuration, one should
exchange IRQ and FIQ in this section. A FIQ can be received during two different
states, either in normal world (``SCR_NS`` is set) or in secure world
(``SCR_NS`` is cleared). When the secure monitor is active (Armv8-A EL3 or
Armv7-A Monitor mode) FIQ is masked. FIQ reception in the two different states
is described below.

**Deliver FIQ to secure world when SCR_NS is set**

When the monitor gets an FIQ exception it:

    1. Saves normal world context and restores secure world context from last
       secure world exit (which will have IRQ and FIQ blocked)
    2. Clears ``SCR_FIQ`` when clearing ``SCR_NS``
    3. Sets “FIQ” as parameter to secure world entry
    4. Does a return from exception into secure context
    5. Secure world unmasks FIQs because of the “FIQ” parameter
    6. FIQ is received as in exception using the state vector
    7. The state vector handle returns from exception in secure world
    8. Secure world issues an SMC to return to normal world
    9. Monitor saves secure world context and restores normal world context
    10. Does a return from exception into restored context

.. figure:: ../images/core/interrupt_handling/fiq.png
    :figclass: align-center

    FIQ received when SCR_NS is set

.. figure:: ../images/core/interrupt_handling/irq_fiq.png
    :figclass: align-center

    FIQ received while processing an IRQ forwarded from secure world

**Deliver FIQ to secure world when SCR_NS is cleared**

Since ``SCR_FIQ`` is cleared when ``SCR_NS`` is cleared a FIQ will be delivered
using the state vector (``VBAR``) in secure world. The FIQ is received as any
other exception by Trusted OS, the monitor is not involved at all.

Trusted thread scheduling
=========================
**Trusted thread for standard services**

OP-TEE standard services are carried through standard SMC. Execution of these
services can be interrupted by foreign interrupts. To suspend and restore the
service execution, optee_os assigns a trusted thread at standard SMCs entry.

The trusted thread terminates when optee_os returns to the normal world with a
service completion status.

A trusted thread execution can be interrupted by a native interrupt. In this
case the native interrupt is handled by the interrupt exception handlers and
once served, optee_os returns to the execution trusted thread.

A trusted thread execution can be interrupted by a foreign interrupt. In this
case, optee_os suspends the trusted thread and invokes the normal world through
the Monitor (optee_os so-called RPC services). The trusted threads will resume
only once normal world invokes the optee_os with the RPC service status.

A trusted thread execution can lead optee_os to invoke a service in normal
world: access a file, get the REE current time, etc. The trusted thread is
suspended/resumed during remote service execution.

**Scheduling considerations**

When a trusted thread is interrupted by a foreign interrupt and when optee_os
invokes a normal world service, the normal world gets the opportunity to
reschedule the running applications. The trusted thread will resume only once
the client application is scheduled back. Thus, a trusted thread execution
follows the scheduling of the normal world caller context.

Optee_os does not implement any thread scheduling. Each trusted thread is
expected to track a service that is invoked from the normal world and should
return to it with an execution status.

The OP-TEE Linux driver (as implemented in `drivers/tee/optee`_ since Linux
kernel 4.12) is designed so that the Linux thread invoking OP-TEE gets assigned
a trusted thread on TEE side. The execution of the trusted thread is tied to the
execution of the caller Linux thread which is under the Linux kernel scheduling
decision. This means trusted threads are scheduled by the Linux kernel.

**Trusted thread constraints**

TEE core handles a static number of trusted threads, see ``CFG_NUM_THREADS``.

Trusted threads are only expensive on memory constrained system, mainly
regarding the execution stack size.

On SMP systems, optee_os can execute several trusted threads in parallel if the
normal world supports scheduling of processes. Even on UP systems, supporting
several trusted threads in optee_os helps normal world scheduler to be
efficient.

----

.. _memory_objects:

Memory objects
**************
A memory object, **MOBJ**, describes a piece of memory. The interface provided
is mostly abstract when it comes to using the MOBJ to populate translation
tables etc. There are different kinds of MOBJs describing:

    - Physically contiguous memory
        - created with ``mobj_phys_alloc(...)``.

    - Virtual memory
        - one instance with the name ``mobj_virt`` available.
        - spans the entire virtual address space.

    - Physically contiguous memory allocated from a ``tee_mm_pool_t *``
        - created with ``mobj_mm_alloc(...)``.

    - Paged memory
        - created with ``mobj_paged_alloc(...)``.
        - only contains the supplied size and makes ``mobj_is_paged(...)``
          return true if supplied as argument.

    - Secure copy paged shared memory
        - created with ``mobj_seccpy_shm_alloc(...)``.
        - makes ``mobj_is_paged(...)`` and ``mobj_is_secure(...)`` return true
          if supplied as argument.

----

.. _mmu:

MMU
***
Translation tables
==================
OP-TEE uses several L1 translation tables, one large spanning 4 GiB and two or
more small tables spanning 32 MiB. The large translation table handles kernel
mode mapping and matches all addresses not covered by the small translation
tables. The small translation tables are assigned per thread and covers the
mapping of the virtual memory space for one TA context.

Memory space between small and large translation table is configured by TTBRC.
TTBR1 always points to the large translation table. TTBR0 points to the a small
translation table when user mapping is active and to the large translation table
when no user mapping is currently active. For details about registers etc,
please refer to a Technical Reference Manual for your architecture, for example
`Cortex-A53 TRM`_.

The translation tables has certain alignment constraints, the alignment (of the
physical address) has to be the same as the size of the translation table. The
translation tables are statically allocated to avoid fragmentation of memory due
to the alignment constraints.

Each thread has one small L1 translation table of its own. Each TA context has a
compact representation of its L1 translation table. The compact representation
is used to initialize the thread specific L1 translation table when the TA
context is activated.

.. graphviz::

    digraph xlat_table {
        graph [
            rankdir = "LR"
        ];
        node [
            fontsize = "16"
            shape = "ellipse"
        ];
        edge [
        ];
        "node_ttb" [
            label = "<f0> TTBR0 | <f1> TTBR1"
            shape = "record"
        ];
        "node_large_l1" [
            label = "<f0> Large L1\nSpans 4 GiB"
            shape = "record"
        ];
        "node_small_l1" [
            label = "Small L1\nSpans 32 MiB\nper entry | <f0> 0 | <f1> 1 | ... | <fn> n"
            shape = "record"
        ];

        "node_ttb":f0 -> "node_small_l1":f0 [ label = "Thread 0 ctx active" ];
        "node_ttb":f0 -> "node_small_l1":f1 [ label = "Thread 1 ctx active" ];
        "node_ttb":f0 -> "node_small_l1":fn [ label = "Thread n ctx active" ];
        "node_ttb":f0 -> "node_large_l1" [ label="No active ctx" ];
        "node_ttb":f1 -> "node_large_l1";
    }


Switching to user mode
======================
This section only applies with following configuration flags:

    - ``CFG_WITH_LPAE=n``
    - ``CFG_CORE_UNMAP_CORE_AT_EL0=y``

When switching to user mode only a minimal kernel mode mapping is kept. This is
achieved by selecting a zeroed out big L1 translation in TTBR1 when
transitioning to user mode. When returning back to kernel mode the original L1
translation table is restored in TTBR1.

Switching to normal world
=========================
When switching to normal world either via a foreign interrupt (see
:ref:`native_foreign_irqs` or RPC there is a chance that secure world will
resume execution on a different CPU. This means that the new CPU need to be
configured with the context of the currently active TA. This is solved by always
setting the TA context in the CPU when resuming execution. Here is room for
improvements since it is more likely than not that it is the same CPU that
resumes execution in secure world.

.. todo::

    Joakim: Jens? Didn't you do some tweaks here already? I.e., "room for
    improvements" above?

----

.. _pager:

Pager
*****
OP-TEE currently requires >256 KiB RAM for OP-TEE kernel memory. This is not a
problem if OP-TEE uses TrustZone protected DDR, but for security reasons OP-TEE
may need to use TrustZone protected SRAM instead. The amount of available SRAM
varies between platforms, from just a few KiB up to over 512 KiB. Platforms with
just a few KiB of SRAM cannot be expected to be able to run a complete TEE
solution in SRAM. But those with 128 to 256 KiB of SRAM can be expected to have
a capable TEE solution in SRAM. The pager provides a solution to this by demand
paging parts of OP-TEE using virtual memory.

Secure memory
=============
TrustZone protected SRAM is generally considered more secure than TrustZone
protected DRAM as there is usually more attack vectors on DRAM. The attack
vectors are hardware dependent and can be different for different platforms.

Backing store
=============
TrustZone protected DRAM or in some cases non-secure DRAM is used as backing
store. The data in the backing store is integrity protected with one hash
(SHA-256) per page (4KiB). Readonly pages are not encrypted since the OP-TEE
binary itself is not encrypted.

Partitioning of memory
======================
The code that handles demand paging must always be available as it would
otherwise lead to deadlock. The virtual memory is partitioned as:

    +--------------+-------------------+
    | Type         | Sections          |
    +==============+===================+
    | unpaged      | | text            |
    |              | | rodata          |
    |              | | data            |
    |              | | bss             |
    |              | | heap1           |
    |              | | nozi            |
    |              | | heap2           |
    +--------------+-------------------+
    | init / paged | | text_init       |
    |              | | rodata_init     |
    +--------------+-------------------+
    | paged        | | text_pageable   |
    |              | | rodata_pageable |
    +--------------+-------------------+
    | demand alloc |                   |
    +--------------+-------------------+

Where ``nozi`` stands for "not zero initialized", this section contains entry
stacks (thread stack when TEE pager is not enabled) and translation tables (TEE
pager cached translation table when the pager is enabled and LPAE MMU is used).

The ``init`` area is available when OP-TEE is initializing and contains
everything that is needed to initialize the pager. After the pager has been
initialized this area will be used for demand paged instead.

The ``demand alloc`` area is a special area where the pages are allocated and
removed from the pager on demand. Those pages are returned when OP-TEE does not
need them any longer. The thread stacks currently belongs this area. This means
that when a stack is not used the physical pages can be used by the pager for
better performance.

The technique to gather code in the different area is based on compiling all
functions and data into separate sections. The unpaged text and rodata is then
gathered by linking all object files with ``--gc-sections`` to eliminate
sections that are outside the dependency graph of the entry functions for
unpaged functions. A script analyzes this ELF file and generates the bits of the
final link script. The process is repeated for init text and rodata. What is
not "unpaged" or "init" becomes "paged".

Partitioning of the binary
==========================
.. note::
    The struct definitions provided in this section are explicitly covered by
    the following dual license:

    .. code-block:: none

        SPDX-License-Identifier: (BSD-2-Clause OR GPL-2.0)

The binary is partitioned into four parts as:


    +----------+
    | Binary   |
    +==========+
    | Header   |
    +----------+
    | Init     |
    +----------+
    | Hashes   |
    +----------+
    | Pageable |
    +----------+

The header is defined as:

.. code-block:: c

    #define OPTEE_MAGIC             0x4554504f
    #define OPTEE_VERSION           1
    #define OPTEE_ARCH_ARM32        0
    #define OPTEE_ARCH_ARM64        1

    struct optee_header {
            uint32_t magic;
            uint8_t version;
            uint8_t arch;
            uint16_t flags;
            uint32_t init_size;
            uint32_t init_load_addr_hi;
            uint32_t init_load_addr_lo;
            uint32_t init_mem_usage;
            uint32_t paged_size;
    };

The header is only used by the loader of OP-TEE, not OP-TEE itself. To
initialize OP-TEE the loader loads the complete binary into memory and copies
what follows the header and the following ``init_size`` bytes to
``(init_load_addr_hi << 32 | init_load_addr_lo)``. ``init_mem_usage`` is used by
the loader to be able to check that there is enough physical memory available
for OP-TEE to be able to initialize at all. The loader supplies in ``r0/x0`` the
address of the first byte following what was not copied and jumps to the load
address to start OP-TEE.

In addition to overall binary with partitions inside described as above, three
extra binaries are generated simultaneously during build process for loaders who
support loading separate binaries:

    +-----------+
    | v2 binary |
    +===========+
    | Header    |
    +-----------+

    +-----------+
    | v2 binary |
    +===========+
    | Init      |
    +-----------+
    | Hashes    |
    +-----------+

    +-----------+
    | v2 binary |
    +===========+
    | Pageable  |
    +-----------+

In this case, loaders load header binary first to get image list and information
of each image; and then load each of them into specific load address assigned in
structure. These binaries are named with `v2` suffix to distinguish from the
existing binaries. Header format is updated to help loaders loading binaries
efficiently:

.. code-block:: c

    #define OPTEE_IMAGE_ID_PAGER    0
    #define OPTEE_IMAGE_ID_PAGED    1

    struct optee_image {
            uint32_t load_addr_hi;
            uint32_t load_addr_lo;
            uint32_t image_id;
            uint32_t size;
    };

    struct optee_header_v2 {
            uint32_t magic;
            uint8_t version;
            uint8_t arch;
            uint16_t flags;
            uint32_t nb_images;
            struct optee_image optee_image[];
    };

Magic number and architecture are identical as original. Version is increased to
two. ``load_addr_hi`` and ``load_addr_lo`` may be ``0xFFFFFFFF`` for pageable
binary since pageable part may get loaded by loader into dynamic available
position. ``image_id`` indicates how loader handles current binary. Loaders who
don't support separate loading just ignore all v2 binaries.

Initializing the pager
======================
The pager is initialized as early as possible during boot in order to minimize
the "init" area. The global variable ``tee_mm_vcore`` describes the virtual
memory range that is covered by the level 2 translation table supplied to
``tee_pager_init(...)``.

Assign pageable areas
---------------------
A virtual memory range to be handled by the pager is registered with a call to
``tee_pager_add_core_area()``.

.. code-block:: c

    bool tee_pager_add_area(tee_mm_entry_t *mm,
                            uint32_t flags,
                            const void *store,
                            const void *hashes);

which takes a pointer to ``tee_mm_entry_t`` to tell the range, flags to tell how
memory should be mapped (readonly, execute etc), and pointers to backing store
and hashes of the pages.

Assign physical pages
---------------------
Physical SRAM pages are supplied by calling ``tee_pager_add_pages(...)``

.. code-block:: c

    void tee_pager_add_pages(tee_vaddr_t vaddr,
                             size_t npages,
                             bool unmap);

``tee_pager_add_pages(...)`` takes the physical address stored in the entry
mapping the virtual address ``vaddr`` and ``npages`` entries after that and uses
it to map new pages when needed. The unmap parameter tells whether the pages
should be unmapped immediately since they does not contain initialized data or
be kept mapped until they need to be recycled. The pages in the "init" area are
supplied with ``unmap == false`` since those page have valid content and are in
use.

Invocation
==========
The pager is invoked as part of the abort handler. A pool of physical pages are
used to map different virtual addresses. When a new virtual address needs to be
mapped a free physical page is mapped at the new address, if a free physical
page cannot be found the oldest physical page is selected instead. When the page
is mapped new data is copied from backing store and the hash of the page is
verified. If it is OK the pager returns from the exception to resume the
execution.

Paging of user TA
=================
Paging of user TAs can optionally be enabled with ``CFG_PAGED_USER_TA=y``.
Paging of user TAs is analogous to paging of OP-TEE kernel parts but with a few
differences:

    - Read/write pages are paged in addition to read-only pages
    - Page tables are managed dynamically

``tee_pager_add_uta_area(...)`` is used to setup initial read/write mapping
needed when populating the TA. When the TA is fully populated and relocated
``tee_pager_set_uta_area_attr(...)`` changes the mapping of the area to strict
permissions used when the TA is running.

----

.. _stacks:

Stacks
******
Different stacks are used during different stages. The stacks are:

    - **Secure monitor stack** (128 bytes), bound to the CPU. Only available if
      OP-TEE is compiled with a secure monitor always the case if the target is
      Armv7-A but never for Armv8-A.

    - **Temp stack** (small ~1KB), bound to the CPU. Used when transitioning
      from one state to another. Interrupts are always disabled when using this
      stack, aborts are fatal when using the temp stack.

    - **Abort stack** (medium ~2KB), bound to the CPU. Used when trapping a data
      or pre-fetch abort. Aborts from user space are never fatal the TA is only
      killed. Aborts from kernel mode are used by the pager to do the demand
      paging, if pager is disabled all kernel mode aborts are fatal.

    - **Thread stack** (large ~8KB), not bound to the CPU instead used by the
      current thread/task. Interrupts are usually enabled when using this stack.

Notes for Armv7-A/AArch32
    .. list-table::
        :header-rows: 1
        :widths: 1 5

        * - Stack
          - Comment

        * - Temp
          - Assigned to ``SP_SVC`` during entry/exit, always assigned to
            ``SP_IRQ`` and ``SP_FIQ``

        * - Abort
          - Always assigned to ``SP_ABT``

        * - Thread
          - Assigned to ``SP_SVC`` while a thread is active

Notes for AArch64
    There are only two stack pointers, ``SP_EL1`` and ``SP_EL0``, available for
    OP-TEE in AArch64. When an exception is received stack pointer is always
    ``SP_EL1`` which is used temporarily while assigning an appropriate stack
    pointer for ``SP_EL0``. ``SP_EL1`` is always assigned the value of
    ``thread_core_local[cpu_id]``. This structure has some spare space for
    temporary storage of registers and also keeps the relevant stack pointers.
    In general when we talk about assigning a stack pointer to the CPU below we
    mean ``SP_EL0``.

Boot
====
During early boot the CPU is configured with the temp stack which is used until
OP-TEE exits to normal world the first time.

Notes for AArch64
    ``SPSEL`` is always ``0`` on entry/exit to have ``SP_EL0`` acting as stack
    pointer.

Normal entry
============
Each time OP-TEE is entered from normal world the temp stack is used as the
initial stack. For fast calls, this is the only stack used. For normal calls an
empty thread slot is selected and the CPU switches to that stack.

Normal exit
===========
Normal exit occurs when a thread has finished its task and the thread is freed.
When the main thread function, ``tee_entry_std(...)``, returns interrupts are
disabled and the CPU switches to the temp stack instead. The thread is freed and
OP-TEE exits to normal world.

RPC exit
========
RPC exit occurs when OP-TEE need some service from normal world. RPC can
currently only be performed with a thread is in running state. RPC is initiated
with a call to ``thread_rpc(...)`` which saves the state in a way that when the
thread is restored it will continue at the next instruction as if this function
did a normal return. CPU switches to use the temp stack before returning to
normal world.

Foreign interrupt exit
======================
Foreign interrupt exit occurs when OP-TEE receives a foreign interrupt. For Arm
GICv2 mode, foreign interrupt is sent as IRQ which is always handled in normal
world. Foreign interrupt exit is similar to RPC exit but it is
``thread_irq_handler(...)`` and ``elx_irq(...)`` (respectively for
Armv7-A/Aarch32 and for Aarch64) that saves the thread state instead. The thread
is resumed in the same way though. For Arm GICv3 mode, foreign interrupt is sent
as FIQ which could be handled by either secure world (EL3 in AArch64) or normal
world. This mode is not supported yet.

Notes for Armv7-A/AArch32
    SP_IRQ is initialized to temp stack instead of a separate stack. Prior to
    exiting to normal world CPU state is changed to SVC and temp stack is
    selected.

Notes for AArch64
    ``SP_EL0`` is assigned temp stack and is selected during IRQ processing. The
    original ``SP_EL0`` is saved in the thread context to be restored when
    resuming.

Resume entry
============
OP-TEE is entered using the temp stack in the same way as for normal entry. The
thread to resume is looked up and the state is restored to resume execution. The
procedure to resume from an RPC exit or an foreign interrupt exit is exactly the
same.

Syscall
=======
Syscall's are executed using the thread stack.

Notes for Armv7-A/AArch32
    Nothing special ``SP_SVC`` is already set with thread stack.

Notes for syscall AArch64
    Early in the exception processing the original ``SP_EL0`` is saved in
    ``struct thread_svc_regs`` in case the TA is executed in AArch64. Current
    thread stack is assigned to ``SP_EL0`` which is then selected. When
    returning ``SP_EL0`` is assigned what is in ``struct thread_svc_regs``. This
    allows ``tee_svc_sys_return_helper(...)`` having the syscall exception
    handler return directly to ``thread_unwind_user_mode(...)``.

----

.. _shared_memory:

Shared Memory
*************
Shared Memory is a block of memory that is shared between the non-secure and the
secure world. It is used to transfer data between both worlds.

Shared Memory Allocation
========================
The shared memory is allocated by the Linux driver from a pool ``struct
shm_pool``, the pool contains:

    - The physical address of the start of the pool
    - The size of the pool
    - Whether or not the memory is cached
    - List of chunk of memory allocated.

.. note::
    - The shared memory pool is physically contiguous.
    - The shared memory area is **not secure** as it is used by both non-secure
      and secure world.

Shared Memory Configuration
===========================
It is the Linux kernel driver for OP-TEE that is responsible for initializing
the shared memory pool, given information provided by the OP-TEE core. The
Linux driver issues a SMC call ``OPTEE_SMC_GET_SHM_CONFIG`` to retrieve the
information

    - Physical address of the start of the pool
    - Size of the pool
    - Whether or not the memory is cached

The shared memory pool configuration is platform specific. The memory mapping,
including the area ``MEM_AREA_NSEC_SHM`` (shared memory with non-secure world),
is retrieved by calling the platform-specific function ``bootcfg_get_memory()``.
Please refer to this function and the area type ``MEM_AREA_NSEC_SHM`` to see the
configuration for the platform of interest. The Linux driver will then
initialize the shared memory pool accordingly.

.. todo::

    Joakim: bootcfg_get_memory(...) is no longer in our code. Text needs update.

Shared Memory Chunk Allocation
==============================
It is the Linux kernel driver for OP-TEE that is responsible for allocating
chunks of shared memory. OP-TEE linux kernel driver relies on linux kernel
generic allocation support (``CONFIG_GENERIC_ALLOCATION``) to allocation/release
of shared memory physical chunks. OP-TEE linux kernel driver relies on linux
kernel dma-buf support (``CONFIG_DMA_SHARED_BUFFER``) to track shared memory
buffers references.

Using shared memory
===================
From the Client Application
    The client application can ask for shared memory allocation using the
    GlobalPlatform Client API function ``TEEC_AllocateSharedMemory(...)``. The
    client application can also provide shared memory through the GlobalPlatform
    Client API function ``TEEC_RegisterSharedMemory(...)``. In such a case, the
    provided memory must be physically contiguous, since OP-TEE core, who does
    not handle scatter-gather memory, is able to use the provided range of
    memory addresses. Note that the reference count of a shared memory chunk is
    incremented when shared memory is registered, and initialized to 1 on
    allocation.

From the Linux Driver
    Occasionally the Linux kernel driver needs to allocate shared memory for the
    communication with secure world, for example when using buffers of type
    ``TEEC_TempMemoryReference``.

From OP-TEE core
    In case OP-TEE core needs information from TEE supplicant (dynamic TA
    loading, REE time request,...), shared memory must be allocated. Allocation
    depends on the use case. OP-TEE core asks for the following shared memory
    allocation:

        - ``optee_msg_arg`` structure, used to pass the arguments to the
          non-secure world, where the allocation will be done by sending a
          ``OPTEE_SMC_RPC_FUNC_ALLOC`` message.

        - In some cases, a payload might be needed for storing the result from
          TEE supplicant, for example when loading a Trusted Application. This
          type of allocation will be done by sending the message
          ``OPTEE_MSG_RPC_CMD_SHM_ALLOC(OPTEE_MSG_RPC_SHM_TYPE_APPL,...)``,
          which then will return:

            - the physical address of the shared memory
            - a handle to the memory, that later on will be used later on when
              freeing this memory.

From TEE Supplicant
    TEE supplicant is also working with shared memory, used to exchange data
    between normal and secure worlds. TEE supplicant receives a memory address
    from the OP-TEE core, used to store the data. This is for example the case
    when a Trusted Application is loaded. In this case, TEE supplicant must
    register the provided shared memory in the same way a client application
    would do, involving the Linux driver.

----

.. _smc:

SMC
***
SMC Interface
=============
OP-TEE's SMC interface is defined in two levels using optee_smc.h_ and
optee_msg.h_. The former file defines SMC identifiers and what is passed in the
registers for each SMC. The latter file defines the OP-TEE Message protocol
which is not restricted to only SMC even if that currently is the only option
available.

SMC communication
=================
The main structure used for the SMC communication is defined in ``struct
optee_msg_arg`` (in optee_msg.h_). If we are looking into the source code, we
could see that communication mainly is achieved using ``optee_msg_arg`` and
``thread_smc_args`` (in thread.h_), where ``optee_msg_arg`` could be seen as the
main structure. What will happen is that the :ref:`linux_kernel` driver will get
the parameters either from :ref:`optee_client` or directly from an internal
service in Linux kernel. The TEE driver will populate the struct
``optee_msg_arg`` with the parameters plus some additional bookkeeping
information.  Parameters for the SMC are passed in registers 1 to 7, register 0
holds the SMC id which among other things tells whether it is a standard or a
fast call.

----

.. _thread_handling:

Thread handling
***************
OP-TEE core uses a couple of threads to be able to support running jobs in
parallel (not fully enabled!). There are handlers for different purposes. In
thread.c_ you will find a function called ``thread_init_primary(...)`` which
assigns ``init_handlers`` (functions) that should be called when OP-TEE core
receives standard or fast calls, FIQ and PSCI calls. There are default handlers
for these services, but the platform can decide if they want to implement their
own platform specific handlers instead.

Synchronization primitives
==========================
OP-TEE has three primitives for synchronization of threads and CPUs:
*spin-lock*, *mutex*, and *condvar*.

Spin-lock
    A spin-lock is represented as an ``unsigned int``. This is the most
    primitive lock. Interrupts should be disabled before attempting to take a
    spin-lock and should remain disabled until the lock is released. A spin-lock
    is initialized with ``SPINLOCK_UNLOCK``.

    .. list-table:: Spin lock functions
        :header-rows: 1
        :widths: 1 5

        * - Function
          - Purpose

        * - ``cpu_spin_lock(...)``
          - Locks a spin-lock

        * - ``cpu_spin_trylock(...)``
          - Locks a spin-lock if unlocked and returns ``0`` else the spin-lock
            is unchanged and the function returns ``!0``

        * - ``cpu_spin_unlock(...)``
          - Unlocks a spin-lock

Mutex
    A mutex is represented by ``struct mutex``. A mutex can be locked and
    unlocked with interrupts enabled or disabled, but only from a normal thread.
    A mutex cannot be used in an interrupt handler, abort handler or before a
    thread has been selected for the CPU. A mutex is initialized with either
    ``MUTEX_INITIALIZER`` or ``mutex_init(...)``.

    .. list-table:: Mutex functions
        :header-rows: 1
        :widths: 1 5

        * - Function
          - Purpose

        * - ``mutex_lock(...)``
          - Locks a mutex. If the mutex is unlocked this is a fast operation,
            else the function issues an RPC to wait in normal world.

        * - ``mutex_unlock(...)``
          - Unlocks a mutex. If there is no waiters this is a fast operation,
            else the function issues an RPC to wake up a waiter in normal world.

        * - ``mutex_trylock(...)``
          - Locks a mutex if unlocked and returns ``true`` else the mutex is
            unchanged and the function returns ``false``.

        * - ``mutex_destroy(...)``
          - Asserts that the mutex is unlocked and there is no waiters, after
            this the memory used by the mutex can be freed.

    When a mutex is locked it is owned by the thread calling ``mutex_lock(...)``
    or ``mutex_trylock(...)``, the mutex may only be unlocked by the thread
    owning the mutex. A thread should not exit to TA user space when holding a
    mutex.

Condvar
    A condvar is represented by ``struct condvar``. A condvar is similar to a
    ``pthread_condvar_t`` in the pthreads standard, only less advanced.
    Condition variables are used to wait for some condition to be fulfilled and
    are always used together a mutex. Once a condition variable has been used
    together with a certain mutex, it must only be used with that mutex until
    destroyed. A condvar is initialized with ``CONDVAR_INITIALIZER`` or
    ``condvar_init(...)``.

    .. list-table:: Condvar functions
        :header-rows: 1
        :widths: 1 5

        * - Function
          - Purpose

        * - ``condvar_wait(...)``
          - Atomically unlocks the supplied mutex and waits in normal world via
            an RPC for the condition variable to be signaled, when the function
            returns the mutex is locked again.

        * - ``condvar_signal(...)``
          - Wakes up one waiter of the condition variable (waiting in
            ``condvar_wait(...)``).

        * - ``condvar_broadcast(...)``
          - Wake up all waiters of the condition variable.

    The caller of ``condvar_signal(...)`` or ``condvar_broadcast(...)`` should
    hold the mutex associated with the condition variable to guarantee that a
    waiter does not miss the signal.

.. _core/arch/arm/kernel/thread.c: https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/kernel/thread.c
.. _optee_msg.h: https://github.com/OP-TEE/optee_os/blob/master/core/include/optee_msg.h
.. _optee_smc.h: https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/include/sm/optee_smc.h
.. _thread.c: https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/kernel/thread.c
.. _thread.h: https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/include/kernel/thread.h

.. _ARM_DEN0028A_SMC_Calling_Convention: http://infocenter.arm.com/help/topic/com.arm.doc.den0028b/ARM_DEN0028B_SMC_Calling_Convention.pdf
.. _Cortex-A53 TRM: http://infocenter.arm.com/help/topic/com.arm.doc.ddi0500j/DDI0500J_cortex_a53_trm.pdf
.. _drivers/tee/optee: https://github.com/torvalds/linux/tree/master/drivers/tee/optee
.. _Trusted Firmware A: https://github.com/ARM-software/arm-trusted-firmware
