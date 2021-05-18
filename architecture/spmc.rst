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

SPMC Program Flow
=================
SP images are stored in the OP-TEE image as early TAs are: the binary images are
embedded in OP-TEE Core in a read-only data section.
This makes it possible to load SPs during boot and no rich-os is needed in the
normal world. ``ldelf`` is used to load and initialise the SPs.

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


Each SP is loaded into the system using ``ldelf`` and started. This is based
around the same process as loading the early TAs.
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
        secure_partition.c -> secure_partition.c:	ldelf_load_ldelf()
        return
        secure_partition.c -> secure_partition.c:	ldlelf_init_with_ldelf()
        return
        secure_partition.c -> secure_partition.c:	sp_init_set_registers()
        return
        return
        secure_partition.c -> secure_partition.c:	enter_sp()
        return
        secure_partition.c -> secure_partition.c:	sp_msg_handler()
        return
        return

:``init_with_ldelf()``:		Load the SP
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


Configuration
=============

Adding SPs to the Image
-----------------------

The following flags have to be enabled to enable the SPMC. The SP images
themself are loaded by using the ``SP_PATHS`` flag.
These should be added to the OP-TEE configuration inside the OP-TEE/build.git 
directory.

.. code-block:: Make

	OPTEE_OS_COMMON_FLAGS += CFG_CORE_FFA=y         # Enable the FF-A transport layer
	OPTEE_OS_COMMON_FLAGS += SP_PATHS="path/to/sp-xxx.elf path/to/sp-yyy.elf" # Add the SPs to the OP-TEE image
	TF_A_FLAGS += SPD=spmd SPMD_SPM_AT_SEL2=0       # Build TF-A with the SPMD enabled and without S-EL2

