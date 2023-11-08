.. _ftrace:

Ftrace (function tracing)
#########################
This section describes how to generate a function call graph for user Trusted
Applications using ``ftrace``. The name comes from the Linux framework which
has a similar purpose, but the OP-TEE ``ftrace`` is very much specific.

A call graph logs all the calls to instrumented functions and contains timing
information. It is therefore a valuable tool to troubleshoot performance
problems or to optimize the code in general.

The configuration option ``CFG_FTRACE_SUPPORT=y`` enables OP-TEE to collect
function graph information from Trusted Applications running in user mode and
compiled with ``-pg``. Once collected, the function graph data is sent to
``tee-supplicant`` via RPC, so they can be saved to disk, later processed
and displayed using helper scripts (``ftrace_format.py`` and ``symbolize.py``
which can be found in ``optee_os/scripts``).

Another configuration option ``CFG_SYSCALL_FTRACE=y`` in addition to
``CFG_FTRACE_SUPPORT=y`` enables OP-TEE to collect function graph information
for syscalls as well while running in kernel mode on behalf of Trusted
Applications. Note that a small number of kernel functions cannot be traced;
they have the ``__noprof`` attribute in the source code.

A third configuration option ``CFG_ULIBS_MCOUNT=y`` enables tracing of user
space libraries contained in ``optee_os`` and used by TAs (such as ``libutee``
and ``libutils``).

Usage
*****

    - Build OP-TEE OS with ``CFG_FTRACE_SUPPORT=y`` and optionally
      ``CFG_ULIBS_MCOUNT=y`` and ``CFG_SYSCALL_FTRACE=y``.

    - Build user TAs with ``-pg``, for instance enable ``CFG_TA_MCOUNT=y`` to
      instrument the whole TA. Also, in case user wants to set ``-pg`` for a
      particular file, following should go in corresponding sub.mk:
      ``cflags-<file-name>-y+=-pg``. Note that instrumented TAs have a larger
      ``.bss`` section. The memory overhead depends on ``CFG_FTRACE_BUF_SIZE``
      macro which can be configured specific to user TAs using config:
      ``CFG_FTRACE_BUF_SIZE=4096`` (default value: 2048, refer to the TA linker
      script for details: ``ta/arch/$(ARCH)/ta.ld.S``).

    - Run the application normally. When the current session exits or there is
      any abort during TA execution, ``tee-supplicant`` will write function
      graph data to ``/tmp/ftrace-<ta_uuid>.out``. If the file already exists,
      a number is appended, such as: ``ftrace-<ta_uuid>.1.out``.

    - Run helper scripts called ``ftrace_format.py`` to translate the function
      graph binary data into human readable text and ``symbolize.py`` to
      convert function addresses into function names:
      ``optee_os/scripts/ftrace_format.py ftrace-<ta_uuid>.out |
      optee_os/scripts/symbolize.py -d <ta_uuid>.elf -d tee.elf``

    - Refer to `commit 5c2c0fb31efb`_ for a full usage example on QEMU.

Typical output
**************

.. code-block:: none

 TEE load address @ 0x5ab04000
 Function graph for TA: cb3e5ba0-adf1-11e0-998b-0002a5d5c51b @ 80085000
             |   1 |  __ta_entry() {
             |   2 |   __utee_entry() {
   43.840 us |   3 |    ta_header_get_session()
    7.216 us |   3 |    tahead_get_trace_level()
   14.480 us |   3 |    trace_set_level()
             |   3 |    malloc_add_pool() {
             |   4 |     raw_malloc_add_pool() {
   46.032 us |   5 |      bpool()
             |   5 |      raw_realloc() {
  166.256 us |   6 |       bget()
   23.056 us |   6 |       raw_malloc_return_hook()
  267.952 us |   5 |      }
  398.720 us |   4 |     }
  426.992 us |   3 |    }
             |   3 |    TEE_GetPropertyAsU32() {
   23.600 us |   4 |     is_propset_pseudo_handle()
             |   4 |     __utee_check_instring_annotation() {
   26.416 us |   5 |      strlen()
             |   5 |      check_access() {
             |   6 |       TEE_CheckMemoryAccessRights() {
             |   7 |        _utee_check_access_rights() {
             |   8 |         syscall_check_access_rights() {
             |   9 |          ts_get_current_session() {
    4.304 us |  10 |           ts_get_current_session_may_fail()
   10.976 us |   9 |          }
             |   9 |          to_user_ta_ctx() {
    2.496 us |  10 |           is_user_ta_ctx()
    8.096 us |   9 |          }
             |   9 |          vm_check_access_rights() {
             |  10 |           vm_buf_is_inside_um_private() {
             |  11 |            core_is_buffer_inside() {
 ...

The duration (function's time of execution) is displayed on the closing bracket
line of a function or on the same line in case the function is the leaf one.
In other words, duration is displayed whenever an instrumented function returns.
It comprises the time spent executing the function and any of its callees. The
Counter-timer Physical Count register (CNTPCT) and the Counter-timer Frequency
register (CNTFRQ) are used to compute durations. Time spent servicing foreign
interrupts is subtracted.

The second column is the stack depth for the current function. It helps
visually match function entries and exit.

.. _commit 5c2c0fb31efb: https://github.com/OP-TEE/optee_os/commit/5c2c0fb31efb
