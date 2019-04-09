.. _ftrace:

Ftrace
######
This section describes how to generate function call graph for user Trusted
Applications using ``ftrace``.

The configuration option ``CFG_TA_FTRACE_SUPPORT=y`` enables OP-TEE to collect
function graph information from Trusted Applications running in user mode and
compiled with ``-pg``. Once collected, the function graph data is formatted
in the ``ftrace.out`` format and sent to ``tee-supplicant`` via RPC, so they
can be saved to disk, later processed and displayed using helper script called
``symbolize.py`` present as part of ``optee_os`` repo.

Usage
*****

    - Build OP-TEE OS and OP-TEE Client with ``CFG_TA_FTRACE_SUPPORT=y``. You
      may also set ``CFG_ULIBS_MCOUNT=y`` in OP-TEE OS to instrument the
      user TA libraries (libutee, libutils, libmpa).

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
      ./optee_os/scripts/symbolize.py -d <ta_uuid>.elf``
