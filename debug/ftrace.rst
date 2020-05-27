.. _ftrace:

Ftrace
######
This section describes how to generate function call graph for user Trusted
Applications using ``ftrace``.

The configuration option ``CFG_FTRACE_SUPPORT=y`` enables OP-TEE to collect
function graph information from Trusted Applications running in user mode and
compiled with ``-pg``. Once collected, the function graph data is formatted
in the ``ftrace.out`` format and sent to ``tee-supplicant`` via RPC, so they
can be saved to disk, later processed and displayed using helper script called
``symbolize.py`` present as part of ``optee_os`` repo.

Another configuration option ``CFG_SYSCALL_FTRACE=y`` in addition to
``CFG_FTRACE_SUPPORT=y`` enables OP-TEE to collect function graph information
for syscalls as well while running in kernel mode on behalf of Trusted
Applications.

Usage
*****

    - Build OP-TEE OS and OP-TEE Client with ``CFG_FTRACE_SUPPORT=y``. You
      may also set ``CFG_ULIBS_MCOUNT=y`` in OP-TEE OS to instrument the
      user TA libraries contained in ``optee_os`` (such as ``libutee`` and
      ``libutils``).

    - Optionally build OP-TEE OS with ``CFG_SYSCALL_FTRACE=y`` to dump
      additional syscall function graph information.

    - Build user TAs with ``-pg``, for instance enable ``CFG_TA_MCOUNT=y`` to
      instrument whole TA. Also, in case user wants to set ``-pg`` for a
      particular file, following should go in corresponding sub.mk:
      ``cflags-<file-name>-y+=-pg``. Note that instrumented TAs have a larger
      ``.bss`` section. The memory overhead depends on ``CFG_FTRACE_BUF_SIZE``
      macro which can be configured specific to user TAs using config:
      ``CFG_FTRACE_BUF_SIZE=4096`` (default value: 2048, refer to the TA linker
      script for details: ``ta/arch/arm/ta.ld.S``).

    - Run the application normally. When the current session exits or there is
      any abort during TA execution, ``tee-supplicant`` will write function
      graph data to ``/tmp/ftrace-<ta_uuid>.out``. If the file already exists,
      a number is appended, such as: ``ftrace-<ta_uuid>.1.out``.

    - Run helper script called ``symbolize.py`` to translate the function graph
      addresses into function names: ``cat ftrace-<ta_uuid>.out |
      ./optee_os/scripts/symbolize.py -d <ta_uuid>.elf -d tee.elf``

Typical output
**************

.. code-block:: bash

              | __ta_entry() {
              |  __utee_entry() {
     1.664 us |   ta_header_get_session();
    11.264 us |   from_utee_params();
      .896 us |   memcpy();
              |   TA_InvokeCommandEntryPoint() {
              |    TEE_GenerateRandom() {
   163.360 us |     utee_cryp_random_number_generate();
   186.848 us |    }
   214.288 us |   }
    19.088 us |   to_utee_params();
              |   ta_header_save_params() {
      .736 us |    memset();
     2.832 us |   }
   304.880 us |  }
   307.168 us | }


The duration (function's time of execution) is displayed on the closing bracket
line of a function or on the same line in case the function is the leaf one.
In other words, duration is displayed whenever an instrumented function returns.
It comprises the time spent executing the function and any of its callees. The
Counter-timer Physical Count register (CNTPCT) and the Counter-timer Frequency
register (CNTFRQ) are used to compute durations. Time spent servicing foreign
interrupts is subtracted.
