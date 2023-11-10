.. sectnum::

====
SPMC
====

This document describes the SPMC (S-EL1) implementation for OP-TEE.
More information on the SPMC can be found in the FF-A specification can be
found in the
`FF-A spec <https://developer.arm.com/documentation/den0077/latest>`_.

.. toctree::
 :numbered:

SPMC Responsibilities
=====================

The SPMC is a critical component in the FF-A flow. Some of its major
responsibilities are:

- Initialisation and run-time management of the SPs:
	The SPMC component is responsible for initialisation of the
	Secure Partitions (loading the image, setting up the stack, heap, ...).
- Routing messages between endpoints:
	The SPMC is responsible for passing FF-A messages from normal world
	to SPs and back. It also responsible for passing FF-A messages between
	SPs.
- Memory management:
	The SPMC is responsible for the memory management of the SPs. Memory
	can be shared between SPs and between a SP to the normal world.

This document describes OP-TEE as a S-EL1 SPMC.

Secure Partitions
=================
Secure Partitions (SPs) are the endpoints used in the FF-A protocol. When
OP-TEE is used as a SPMC SPs run primarily inside S-EL0.

OP-TEE will use FF-A for it transport layer when the OP-TEE ``CFG_CORE_FFA=y``
configuration flag is enabled.
The SPMC will expose the OP-TEE core, privileged mode, as an secure endpoint
itself. This is used to handle all GlobalPlaform programming mode operations.
All GlobalPlatform messages are encapsulated inside FF-A messages.
The OP-TEE endpoint will unpack the messages and afterwards handle them as
standard OP-TEE calls. This is needed as TF-A (S-EL3) does only allow
FF-A messages to be passed to the secure world when the SPMD is enabled.

SPs run from the initial boot of the system until power down and don't have any
built-in session management compared to GPD TEE TAs. The only means of
communicating with the outside world is through messages defined in the FF-A
specification. The context of a SP is saved between executions.

The
`Trusted Service <https://www.trustedfirmware.org/projects/trusted-services/>`_
repository includes the libsp libary which export all needed functions to build
a S-EL0 SP. It also includes many examples of how to create and implement a SP.

Secure Partition formats
========================

OP-TEE specific ELF format
--------------------------

OP-TEE uses an ELF format for its :ref:`trusted_applications`. It has an OP-TEE
specific section which contains a header structure for describing the Trusted
Application. A very similar format can be used for Secure Partitions. The same
ELF format allows OP-TEE to use the built-in ELF loader (``ldelf``) with all its
features like handling relocations or ASLR. In this case a different section is
used for the header structure to distinguish between Trusted Applications and
Secure Partitions.

SPMC agnostic flat binary format
--------------------------------

This simple binary format aims for maximum portability between SPMC
implementations by removing the dependency on an ELF loader and implementation
specific metadata in the SP image. The SPMC can simply copy the binary into the
memory and start running it. The relocations, the stack setup and any further
initialization steps should be handled by the startup code of the secure
partition. The access rights for different sections of the binary can be
configured either by adding load relative memory regions to the SP manifest or
by using the ``FFA_MEM_PERM_SET`` interface in the startup code.

SPMC Program Flow
=================
SP images are either embedded into the OP-TEE image or loaded from the FIP by
BL2. This makes it possible to start SPs during boot, before the rich OS is
available in the normal world.

Starting SPs
------------
SPs are loaded and started as the last step in OP-TEE's initialisation process.
This is done by adding ``sp_init_all()`` to the ``boot_final`` initcall level.

.. uml::
        :width: 800

        autoactivate on
        thread_optee_smc_a64.s -> boot_final
        boot_final -> secure_partition.c:			sp_init_all()
        loop for each SP
        secure_partition.c -> secure_partition.c:		sp_init_uuid()
        return
        end
        return
        return
        autoactivate off
        thread_optee_smc_a64.s -> thread_optee_smc_a64.s:	thread_ffa_msg_wait()
        thread_optee_smc_a64.s -> thread_optee_smc_a64.s:	ffa_msg_loop()
        autoactivate off
        thread_optee_smc_a64.s -> SPMD:				SMC

:``sp_init_all()``:		Initialise all SPs which have been added by the
				``SP_PATHS`` compiler option and run them
:``thread_ffa_msg_wait()``:	All SPs are loaded and started. A
				``FFA_MSG_WAIT`` message is sent to the Normal
				World.


Each ELF format SP is loaded into the system using ``ldelf`` and started. This
is based around the same process as loading the early TAs.
For each binary format SP a simpler method is used to copy the binary into a
suitable memory area.
All SPs are run after they are loaded and run until a ``FFA_MSG_WAIT`` is sent
by the SP.


.. uml::

        autoactivate on
        secure_partition.c -> secure_partition.c:	sp_init_uuid()
        secure_partition.c -> secure_partition.c:	sp_open_session()
        secure_partition.c -> secure_partition.c:	find_sp()
        return
        secure_partition.c -> secure_partition.c:	sp_create_session()
        return
        alt OP-TEE specific ELF format
                secure_partition.c -> secure_partition.c:ldelf_load_ldelf()
                return
                secure_partition.c -> secure_partition.c:ldlelf_init_with_ldelf()
                return
        else SPMC agnostic flat binary format
                secure_partition.c -> secure_partition.c:load_binary_sp()
                return
        end
        secure_partition.c -> secure_partition.c:	sp_init_set_registers()
        return
        return
        secure_partition.c -> secure_partition.c:	enter_sp()
        return
        secure_partition.c -> secure_partition.c:	sp_msg_handler()
        return
        return

:``init_with_ldelf()``:		Load the OP-TEE specific ELF format SP
:``load_binary_sp()``:		Load the SPMC agnostic flat binary format SP
:``sp_init_info()``:		Initialise the ``struct ffa_init_info``. The
				``struct ffa_init_info`` is passed to the SP
				during it first run.
:``sp_init_set_registers()``:	Initialise the registers of the SP
:``sp_msg_handler()``:		Handle the SPs FF-A message

Once all SPs are loaded and started we return to the SPMD and the Normal World
is booted.


SP message handling
-------------------

The SPMC is split into 2 main message handlers:

:``thread_spmc_msg_recv()`` thread_spmc.c:	Used to handle message coming
						from the Normal World.
:``sp_msg_handler()`` spmc_sp_handler.c:	Used to handle message where
						the source or the destination
						is a SP.

When a ``FFA_MSG_SEND_DIRECT_REQ`` message is received by the SPMC from the
Normal World, a new thread is started.
The FF-A message is passed to the thread and it will call the
``sp_msg_handler()`` function.

Whenever the SPMC (``sp_msg_handler()``) receives a message not intended for
one of the SPs, it will exit the thread and return to the Normal World
passing the FF-A message.

Currently only a ``FFA_MSG_SEND_DIRECT_REQ`` can be passed from the Normal
World to a SP.

.. uml::
        :width: 800

        skinparam backgroundcolor transparent
        participant "None-secure world" as None_secure_world

        box "S-EL3"
        participant SPMD
        end box

        box "S-EL1"
        participant thread_spmc
        participant spmc_sp_handler
        end box


        box "S-EL0"
        participant SP
        end box

        autoactivate on
        None_secure_world -> SPMD: FFA_MSG_SEND_DIRECT_REQ \n <font color=red>SMC
        SPMD -> thread_spmc: thread_spmc_msg_recv() \n <font color=red>ERET

        thread_spmc -> spmc_sp_handler : spmc_sp_start_thread()
        == thread ==
        spmc_sp_handler -> spmc_sp_handler : sp_msg_handler()

        loop FF-A dst != NSW
                spmc_sp_handler -> spmc_sp_handler: ffa_handle_sp_direct_req()
                spmc_sp_handler -> spmc_sp_handler: enter_sp()
                spmc_sp_handler -> spmc_sp_handler: sp_enter_invoke_cmd()
                autoactivate off
                spmc_sp_handler -> SP: __thread_enter_user_mode() \n <font color=red>ERET
                activate SP

                SP -> spmc_sp_handler : (FF-A message) sp_handle_svc() \n <font color=red>SVC
                deactivate SP
                activate spmc_sp_handler
                spmc_sp_handler -> spmc_sp_handler: Store SP context
                spmc_sp_handler -> spmc_sp_handler: return to sp_enter_invoke_cmd()
                deactivate spmc_sp_handler
                spmc_sp_handler -> spmc_sp_handler: Retrieve FF-A message from SP context
                return
                return
                return
        end
                return
        == End of thread ==
                return
        return \n <font color=red>SMC
        return FFA_MSG_SEND_DIRECT_RESP \n <font color=red>ERET


Every message received by the SPMC from the Normal World is handled in the
``thread_spmc_msg_recv()`` function.

When entering a SP we need to be running in a OP-TEE thread. This is needed to
be able to push the TS session (We push the TS session to get access to the SP
memory).
Currently the only possibility to enter a SP from the Normal world is via a
``FFA_MSG_SEND_DIRECT_REQ``. Whenever we receive a ``FFA_MSG_SEND_DIRECT_REQ``
message which doesn't have OP-TEE as the endpoint-id, we start a thread and
forward the FF-A message to the ``sp_msg_handler()``.

The ``sp_msg_handler()`` is responsible for all messages coming or going
to/from a SP. It runs in a while loop and will handle every message until it
comes across a messages which is not intended for the secure world.
After a message is handled by the SPMC or when it needs to be forwarded to a SP,
``sp_enter()`` is called.
``sp_enter()`` will copy the FF-A arguments and resume the SP.

When the SPMC needs to have access to the SPs memory, it will call
``ts_push_current_session()`` to gain access and ``ts_pop_current_session()``
to release the access.

Running and exiting SPs
-----------------------

The SPMC resumes/starts the SP by calling the ``sp_enter()``. This will set up
the SP context and jump into S-EL0.
Whenever the SP performs a system call it will end up in ``sp_handle_svc()``.
``sp_handle_svc()`` stores the current context of the SP and makes sure that we
don't return to S-EL0 but instead returns to S-EL1 back to ``sp_enter()``.
``sp_enter()`` will pass the FF-A registers (x0-x7) to
``spmc_sp_msg_handler()``. This will process the FF-A message.


RxTx buffer managment
---------------------
RxTx buffers are used by the SPMC to exchange information between an endpoint
and the SPMC. The rxtx_buf struct is used by the SPMC for abstracting buffer
management.
Every SP has a ``struct rxtx_buf`` wich will be passed to every function that
needs access to the rxtx buffer.
A separate ``struct rxtx_buf`` is defined for the Normal World, which gives
access to the Normal World buffers.

FF-A compliance
===============

.. |ffa_fs| replace:: :opticon:`check-circle-fill`
.. |ffa_ps| replace:: :opticon:`check-circle`
.. |ffa_ns| replace:: :opticon:`x`
.. |ffa_na| replace:: :opticon:`horizontal-rule`

Legend
------

* |ffa_fs| Fully supported
* |ffa_ps| Partially implemented
* |ffa_ns| Not supported
* |ffa_na| Does not apply for the FF-A instance or version

Partition boot protocol
-----------------------

Only FF-A v1.0 partition boot protocol is supported by the SPMC.

Supported partition manifest fields
-----------------------------------

+--------------------------------+-----------+-----------+-----------+
| Field                          | Mandatory | FF-A v1.0 | FF-A v1.1 |
+================================+===========+===========+===========+
| FF-A version                   | Yes       | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| UUID                           | Yes       | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Partition ID                   | No        | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Auxiliary IDs                  | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Name (description)             | No        | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Number of execution contexts   | Yes       | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Run-time EL                    | Yes       | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Execution state                | Yes       | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Load address                   | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Entry point offset             | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Translation granule            | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Boot order                     | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| RX/TX information              | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Messaging method               | Yes       | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Primary scheduler implemented  | No        | |ffa_na|  | |ffa_na|  |
+--------------------------------+-----------+-----------+-----------+
| Run-time model                 | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Tuples                         | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
|                        **Memory regions**                          |
+--------------------------------+-----------+-----------+-----------+
| Base address                   | No        | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Load address relative offset   | No        | |ffa_na|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Page count                     | Yes       | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Attributes                     | Yes       | |ffa_ps|  | |ffa_ps|  |
+--------------------------------+-----------+-----------+-----------+
| Name                           | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Stream & SMMU IDs              | No        | |ffa_na|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Stream ID access permissions   | No        | |ffa_na|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
|                        **Device regions**                          |
+--------------------------------+-----------+-----------+-----------+
| Physical base address          | Yes       | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Page count                     | Yes       | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Attributes                     | Yes       | |ffa_fs|  | |ffa_fs|  |
+--------------------------------+-----------+-----------+-----------+
| Interrupts                     | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| SMMU IDs                       | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Stream IDs                     | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Exclusive access and ownership | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+
| Name                           | No        | |ffa_ns|  | |ffa_ns|  |
+--------------------------------+-----------+-----------+-----------+

Limitations
^^^^^^^^^^^

* The values of mandatory but not supported fields are ignored by the SP loader.
  This means all values are accepted but the SPMC might behave differently than
  expected.
* Memory region attributes doesn't support shareability and cacheability flags.

Supported FF-A interfaces
-------------------------

The table below describes the implementation level of each FF-A interface on
different FF-A instances. The two instances are between OP-TEE SPMC and the SPMC
and between OP-TEE SPMC and its S-EL0 secure partitions. The FF-A specification
uses 'Secure Phyisical' and 'Secure Virtual' terms for these instances.

+--------------------------+-----------------------+-----------------------+
|                          | OP-TEE <-> SPMD       | OP-TEE <-> S-EL0 SPs  |
| Interface                +-----------+-----------+-----------+-----------+
|                          | FF-A v1.0 | FF-A v1.1 | FF-A v1.0 | FF-A v1.1 |
+==========================+===========+===========+===========+===========+
| FFA_ERROR                | |ffa_fs|  | |ffa_ps|  | |ffa_fs|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_SUCCESS              | |ffa_fs|  | |ffa_fs|  | |ffa_ps|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_INTERRUPT            | |ffa_ps|  | |ffa_ps|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_VERSION              | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_FEATURES             | |ffa_ps|  | |ffa_ns|  | |ffa_ps|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_RX_ACQUIRE           | |ffa_na|  | |ffa_ns|  | |ffa_na|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_RX_RELEASE           | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_RXTX_MAP             | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_RXTX_UNMAP           | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_PARTITION_INFO_GET   | |ffa_fs|  | |ffa_ns|  | |ffa_fs|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_ID_GET               | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_SPM_ID_GET           | |ffa_na|  | |ffa_ns|  | |ffa_na|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MSG_WAIT             | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_YIELD                | |ffa_na|  | |ffa_ns|  | |ffa_na|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_RUN                  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_NORMAL_WORLD_RESUME  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MSG_SEND             | |ffa_na|  | |ffa_na|  | |ffa_na|  | |ffa_na|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MSG_SEND2            | |ffa_na|  | |ffa_ns|  | |ffa_na|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MSG_SEND_DIRECT_REQ  | |ffa_fs|  | |ffa_ps|  | |ffa_fs|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MSG_SEND_DIRECT_RESP | |ffa_fs|  | |ffa_ps|  | |ffa_fs|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MSG_POLL             | |ffa_na|  | |ffa_na|  | |ffa_na|  | |ffa_na|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_DONATE           | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_LEND             | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_SHARE            | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_RETRIEVE_REQ     | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_RETRIEVE_RESP    | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_RELINQUISH       | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  | |ffa_ps|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_RECLAIM          | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_PERM_GET         | |ffa_na|  | |ffa_na|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_PERM_SET         | |ffa_na|  | |ffa_na|  | |ffa_fs|  | |ffa_fs|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_FRAG_RX          | |ffa_fs|  | |ffa_fs|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_FRAG_TX          | |ffa_fs|  | |ffa_fs|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_OP_PAUSE         | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+
| FFA_MEM_OP_RESUME        | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  | |ffa_ns|  |
+--------------------------+-----------+-----------+-----------+-----------+

Limitations
^^^^^^^^^^^

* FF-A v1.1 error code ``NO_DATA`` is not supported.
* ``FFA_SUCCESS`` is not supported as a response to an
  ``FFA_MSG_SEND_DIRECT_REQ`` message.
* Non-secure interrupts are not forwarded to the normal world via
  ``FFA_INTERRUPT``.
* Interrupts cannot be forwarded to S-EL0 secure partitions.
* Only ``FFA_RXTX_MAP`` feature query is supported by the ``FFA_FEATURES``
  interface. ``FFA_MEM_DONATE``, ``FFA_MEM_LEND``, ``FFA_MEM_SHARE`` and
  ``FFA_MEM_RETRIEVE_REQ`` feature query is not implemented.
* FF-A v1.1 ``Flags`` field in ``FFA_MSG_SEND_DIRECT_REQ`` and
  ``FFA_MSG_SEND_DIRECT_RESP`` calls is not supported.
* Transferring memory transaction descriptors in a buffer distinct from the TX
  buffer is not supported by the secure virtual instance.
* Transferring fragmented memory transaction descriptors is not supported by the
  secure virtual instance.
* The only supported 'Memory region attributes descriptor' value is normal
  memory, write-back cacheability and inner shareable. All other values are
  denied on the secure physical instance. The secure virtual instance's
  implementation ignores the value of this descriptor but uses the same
  attributes for the region.
* The NS flag support in not implemented for 'Memory region attributes
  descriptor'.
* Only read-write non-executable value can be used in the 'Memory access
  permissions descriptor' at the secure phyisical instance.
* The ``Flags`` field of ``FFA_MEM_RELINQUISH`` is ignored.
* The secure phyisical instanced doesn't implemented the receiving of
  ``FFA_MEM_RELINQUISH``.
* Time slicing of memory management operations is not supported.

Configuration
=============

SPMC config options
---------------------------

To configure OP-TEE as a S-EL1 SPMC with Secure Partition support, the following
flags should be set for optee_os:

- ``CFG_CORE_SEL1_SPMC=y``
- ``CFG_SECURE_PARTITION=y``
- ``CFG_DT=y``
- ``CFG_MAP_EXT_DT_SECURE=y``

Furthermore TF-A should be configured as the SPMD, expecting a S-EL1 SPMC:

- ``SPD=spmd``
- ``SPMD_SPM_AT_SEL2=0``
- ``ARM_SPMC_MANIFEST_DTS=<path to SPMC manifest dts>``

SP loading mechanism
---------------------

OP-TEE SPMC supports two methods for finding and loading the SP executable
images. Currently only ELF executables are supported. In the build repo the
loading method can be selected with the SP_PACKAGING_METHOD option.

Embedded SP
^^^^^^^^^^^

In this case the early TA mechanism of optee_os is reused: the SP ELF files are
embedded into the main OP-TEE binary. Each ELF should start with a specific
section (.sp_head) containing a struct which describes the SP (UUID, stack size,
etc.). The images can be added to optee_os using the ``SP_PATHS`` config option,
the build repo will set this up automatically when
``SP_PACKAGING_METHOD=embedded`` is selected. The images passed in ``SP_PATHS``
are processed by ``ts_bin_to_c.py`` in optee_os and linked into the main binary.
At runtime the ``for_each_secure_partition()`` macro can iterate through these
images, so a particular SP can be found by UUID and then loaded.

The SP manifest file `[1]`_ used by the SPMC to setup SPs is also handled by
``ts_bin_to_c.py``, it will be concatenated to the end of the SP ELF.

FIP SP
^^^^^^

In this case the SP ELF files and the corresponding SP manifest DTs are
encapsulated into SP packages and packed into the FIP. The goal of providing
this alternative flow is to make updating SPs easier (independent of the main
OP-TEE binary) and to get aligned with Hafnium (S-EL2 SPMC). For more
information about the FIP, please refer to the TF-A documentation `[2]`_. The SP
packaging process and the package format is provided by TF-A, detailed
description is available at `[3]`_. In the build repo this method can be
selected by ``SP_PACKAGING_METHOD=fip``, it covers all the necessary setup
automatically. In case of using another buildsystem, the following steps should
be implemented:

- TF-A config ``SP_LAYOUT_FILE``: provide a JSON file which describes the SPs
  (path to SP executable and corresponding DT, example `[4]`_). The TF-A
  buildsystem will create the SP packages (using sptool) based on this and pack
  them into the FIP.

- TF-A config ``ARM_BL2_SP_LIST_DTS``: provide a DT snippet which describes the
  SPs' UUIDs and load addresses (example: `[5]`_). This will be injected into
  the SP list in ``TB_FW_CONFIG`` DT of TF-A, and BL2 will load the SP packages
  based on this. Note that BL2 doesn't automatically load all images from the
  FIP: it's necessary to explicitly define them in ``TB_FW_CONFIG`` (using this
  injected snippet or manually editing the DT).

- TF-A config ``ARM_SPMC_MANIFEST_DTS``: provide the SPMC manifest (example:
  `[6]`_). This DT is passed to the SPMC as a boot argument (in the TF-A naming
  convention this is the ``TOS_FW_CONFIG``). It should contain the list of SP
  packages and their load addresses in the ``compatible = "arm,sp_pkg"`` node.

At boot optee_os will parse the SP package load addresses from the SPMC manifest
and find the SP packages already loaded by BL2. Iterating through the SP
packages, based on the SP package header in each package it will map the SP
executable image and the corresponding manifest DT and collect these to the
``fip_sp_list`` list. Later when initialising the SPs, the ``for_each_fip_sp``
macro is used to iterate this list and load the executables, just like for the
embedded SP case.

.. _[1]:

[1] https://trustedfirmware-a.readthedocs.io/en/v2.6/components/ffa-manifest-binding.html

.. _[2]:

[2] https://trustedfirmware-a.readthedocs.io/en/v2.6/design/firmware-design.html#firmware-image-package-fip

.. _[3]:

[3] https://trustedfirmware-a.readthedocs.io/en/v2.6/components/secure-partition-manager.html#secure-partition-packages

.. _[4]:

[4] https://trustedfirmware-a.readthedocs.io/en/v2.6/components/secure-partition-manager.html#describing-secure-partitions

.. _[5]:

[5] https://github.com/OP-TEE/build/blob/master/fvp/bl2_sp_list.dtsi

.. _[6]:

[6] https://github.com/OP-TEE/build/blob/master/fvp/spmc_manifest.dts
