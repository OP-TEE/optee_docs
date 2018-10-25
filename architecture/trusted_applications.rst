.. _trusted_applications:

####################
Trusted Applications
####################
There are two ways to implement Trusted Applications (TAs), Pseudo TAs and user
mode TAs. User mode TAs are full featured Trusted Applications as specified by
the :ref:`globalplatform_api` TEE specifications, these are simply the ones
people are referring to when they are saying "Trusted Applications" and in most
cases this is the preferred type of TA to write and use.

.. _pta:

Pseudo Trusted Applications
***************************
These are implemented directly to the OP-TEE core tree in, e.g.,
``core/arch/arm/pta`` and are built along with and statically built into the
OP-TEE core blob.

The Pseudo Trusted Applications included in OP-TEE already are OP-TEE secure
privileged level services hidden behind a "GlobalPlatform TA Client" API. These
Pseudo TAs are used for various purposes such as specific secure services or
embedded tests services.

Pseudo TAs **do not** benefit from the GlobalPlatform Core Internal API support
specified by the GlobalPlatform TEE specs. These APIs are provided to TAs as a
static library each TA shall link against (the ":ref:`libutee`") and that calls
OP-TEE core service through system calls. As OP-TEE core does not link with
:ref:`libutee`, Pseudo TAs can **only** use the OP-TEE core internal APIs and
routines.

As Pseudo TAs runs at the same privileged execution level as the OP-TEE core
code itself and that might or might not be desirable depending on the use case.

In most cases an unprivileged (user mode) TA is the best choice instead of
adding your code directly to the OP-TEE core. However if you decide your
application is best handled directly in OP-TEE core like this, you can look at
``core/arch/arm/pta/stats.c`` as a template and just add your Pseudo TA based on
that to the ``sub.mk`` in the same directory.

.. _user_mode_ta:

User Mode Trusted Applications
******************************
User Mode Trusted Applications are loaded (mapped into memory) by OP-TEE core in
the Secure World when something in Rich Execution Environment (REE) wants to
talk to that particular application UUID. They run at a lower CPU privilege
level than OP-TEE core code. In that respect, they are quite similar to regular
applications running in the REE, except that they execute in Secure World.

Trusted Application benefit from the GlobalPlatform :ref:`tee_internal_core_api`
as specified by the GlobalPlatform TEE specifications. There are several types
of user mode TAs, which differ by the way they are stored.

TA locations
************
Plain TAs (user mode) can reside and be loaded from various places. There are
three ways currently supported in OP-TEE.

.. _early_ta:

Early TA
========
The so-called early TAs are virtually identical to the REE FS TAs, but instead
of being loaded from the Normal World file system, they are linked into a
special data section in the TEE core blob. Therefore, they are available even
before ``tee-supplicant`` and the REE's filesystems have come up. Please find
more details in the `early TA commit`_.

.. _ree_fs_ta:

REE filesystem TA
=================
They consist of a cleartext signed ELF_ file, named from the UUID of the TA and
the suffix ``.ta``. They are built separately from the OP-TEE core boot-time
blob, although when they are built they use the same build system, and are
signed with the key from the build of the original OP-TEE core blob.

Because the TAs are signed, they are able to be stored in the untrusted REE
filesystem, and ``tee-supplicant`` will take care of passing them to be checked
and loaded by the Secure World OP-TEE core. Note that this type of TA isn't
encrypted.

.. _secure_storage_ta:

Secure Storage TA
=================
These are stored in secure storage. The meta data is stored in a database of all
installed TAs and the actual binary is stored encrypted and integrity protected
as a separate file in the untrusted REE filesystem (flash). Before these TAs can
be loaded they have to be installed first, this is something that can be done
during initial deployment or at a later stage.

For test purposes the test program xtest can install a TA into secure storage
with the command:

.. code-block:: bash

    $ xtest --install-ta


.. _ta_properties:

TA Properties
*************
This section give a more in depth description of the TA properties (see
:ref:`build_trusted_applications` also).

1. GlobalPlatform properties
============================
Standard TA properties must be defined through property flag in macro
``TA_FLAGS`` in ``user_ta_header_defines.h``

1.1 Single Instance
===================
``"gpd.ta.singleInstance"`` is a boolean property of the TA. This property
defines if one instance of the TA must be created and will receive all open
session request, or if a new specific TA instance must be created for each
incoming open session request. OP-TEE TA flag ``TA_FLAG_SINGLE_INSTANCE`` sets
to configuration of this property. The boolean property is set to ``true`` if
``TA_FLAGS`` sets bit ``TA_FLAG_SINGLE_INSTANCE``, otherwise the boolean
property is set to ``false``.

1.2 Multi-session
=================
``"gpd.ta.multiSession"`` is a boolean property of the TA. This property defines
if the TA instance can handle several sessions. If disabled, TA instance support
only one session. In such case, if the TA already has a opened session, any open
session request will return with a busy error status.

.. note::

    This property is **meaningless** if TA is **NOT** SingleInstance TA.

OP-TEE TA flag ``TA_FLAG_MULTI_SESSION`` sets to configuration of this property.
The boolean property is set to ``true`` if ``TA_FLAGS`` sets bit
``TA_FLAG_MULTI_SESSION``, otherwise the boolean property is set to ``false``.

1.3 Keep Alive
==============
``"gpd.ta.instanceKeepAlive"`` is a boolean property of the TA. This property
defines if the TA instance created must be destroyed or not when all sessions
opened towards the TA are closed. If the property is enabled, TA instance, once
created (at 1st open session request), is never removed unless the TEE itself is
restarted (boot/reboot).

.. note::

    This property is **meaningless** if TA is **NOT** SingleInstance TA.

OP-TEE TA flag ``TA_FLAG_INSTANCE_KEEP_ALIVE`` sets to configuration of this
property. The boolean property is set to ``true`` if ``TA_FLAGS`` sets bit
``TA_FLAG_INSTANCE_KEEP_ALIVE``, otherwise the boolean property is set to
``false``.

1.4 Heap Size
=============
``"gpd.ta.dataSize"`` is a 32bit integer property of the TA. This property
defines the size in bytes of the TA allocation pool, in which ``TEE_Malloc()``
and friends allocate memory. The value of the property must be defined by the
macro ``TA_DATA_SIZE`` in ``user_ta_header_defines.h`` (see
:ref:`build_ta_properties`).

1.5 Stack Size
==============
``"gpd.ta.stackSize"`` is a 32bit integer property of the TA. This property
defines the size in bytes of the stack used for TA execution. The value of the
property must be defined by the macro ``TA_STACK_SIZE`` in
``user_ta_header_defines.h`` (see :ref:`build_ta_properties`).

2. Property extensions
======================
2.1 User Mode Flag
==================
``TA_FLAG_USER_MODE`` is a bit flag supported by ``TA_FLAGS``. This property
flag is currently meaningless in OP-TEE. It may be set or not without impact on
TA execution. All OP-TEE TAs are executed in user mode/level. Because of this we
**do not** recommend to use this flag.

2.2 DDR Flag
============
``TA_FLAG_EXEC_DDR`` is a bit flag supported by ``TA_FLAGS``. This property flag
is currently meaningless in OP-TEE. Nevertheless it shall be set. It is a legacy
property flag that aimed at targeting location for the TA execution, internal
RAM or external DDR. Therefore all TAs must set ``TA_FLAG_EXEC_DDR`` in
``TA_FLAGS`` in their ``user_ta_header_defines.h`` header file (see
:ref:`user_ta_header_defines_h`).

.. note::

    This flag will soon be deprecated.

2.3 Secure Data Path Flag
=========================
``TA_FLAG_SECURE_DATA_PATH`` is a bit flag supported by ``TA_FLAGS``. This
property flag claims the secure data support from the OP-TEE OS for the TA.
Refer to the OP-TEE OS for secure data path support. TAs that do not set
``TA_FLAG_SECURE_DATA_PATH`` in the value of ``TA_FLAGS`` will **not** be able
to handle memory reference invocation parameters that relate to secure data path
buffers.

2.4 Remap Support Flag
======================
``TA_FLAG_REMAP_SUPPORT`` is a bit flag supported by ``TA_FLAGS``. This property
flag is currently meaningless in OP-TEE and therefore we recommend to not use
this flag.

.. note::

    This flag will soon be deprecated.

2.5 Cache maintenance Flag
==========================
``TA_FLAG_CACHE_MAINTENANCE`` is a bit flag supported by ``TA_FLAGS``. This
property flag claims access to the cache maintenance API for the TA:
``TEE_CacheXxxx()``. Refer to the OP-TEE to check if cache API support is
enabled. TAs that do not set ``TA_FLAG_CACHE_MAINTENANCE`` in the value of their
``TA_FLAGS`` will not be able to call the cache maintenance API.

.. _ELF: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
.. _early TA commit: https://github.com/OP-TEE/optee_os/commit/d0c636148b3a
