.. _libraries:

#########
Libraries
#########

.. _libutils:

libutils
********

OP-TEE core and OP-TEE development kit for Trusted Application provide a
standard C library that is named **libutils**. It implements many
standard functions like ``snprintf()``, ``strncmp()``, ``memcpy()``,
``malloc()``. ``qsort()``, and many more but not all standard C library
functions.

Note however that Trusted Applications implemented in C should use GP TEE
Internal Core API functions rather than their standard C library function
equivalent (e.g. ``TEE_MemMove()`` instead of ``memcpy()`` and
``memmove()``, or ``TEE_Malloc()`` instead of ``malloc()`` and friends).
This makes those TAs implementation more portable to other GP
TEE compliant environments.

When ``CFG_ULIBS_SHARED`` is enabled, **libutils** is assigned UUID
**71855bba-6055-4293-a63f-b0963a737360**.

.. _libutee:

libutee
*******
The :ref:`tee_internal_core_api` describes services that are provided to Trusted
Applications. **libutee** is a library that implements this API.

libutee is designed as a userland library specifically dedicated to OP-TEE
Trusted Applications and aims at being executed in the non-privileged secure
userspace.

Some services for this API are fully statically implemented inside the libutee
library while some services for the API are implemented inside the OP-TEE core
(privileged level) and libutee calls such services through system calls.

When ``CFG_ULIBS_SHARED`` is enabled, **libutee** is assigned UUID
**4b3d937e-d57e-418b-8673-1c04f2420226**.

libmbedtls
**********

OP-TEE OS source tree provides support of the Mbed TLS library, named
**libmbedtls**.

A specific build sequence can compile an instance of **libmbedtls** and link
it to OP-TEE core. Another build sequence compiles an instance of
**libmbedtls** that can be linked with Trusted Applications.

When Mbed TLS is embedded in OP-TEE core, it is used as the default software
implementation for most cryptography operations. When so, **libtomcrypt** is
still used as default software implementation for few crypto operations.
Embedding Mbed TLS in OP-TEE core requires ``CFG_CRYPTOLIB_NAME=mbedtls``
and ``CFG_CRYPTOLIB_DIR=core/lib/libmbedtls``.

When ``CFG_ULIBS_SHARED`` is enabled, **libmbedtls** userland library is
assigned UUID **87bb6ae8-4b1d-49fe-9986-2b966132c309**.

libunw
******

OP-TEE OS source tree implements execution stack back trace debug facilities
available to both OP-TEE core and Trusted Applications. The feature relies
on a library named **libunw**.

**libunw**, when linked to a Trusted Application, is always linked as a static
library.

libdl
*****

**libdl** library implement API function ``dlopen()``, ``dlsym()`` and
``dlclose()`` used by Trusted Applications to support dynamic shared libraries.

When ``CFG_ULIBS_SHARED`` is enabled, **libdl** is assigned UUID
**be807bbd-81e1-4dc4-bd99-3d363f240ece**.

.. _statci_or_shared_lib:

Static vs Shared libraries
**************************

OP-TEE core supports only static libraries that are linked at build time to
produce the monolithic OP-TEE core image.

OP-TEE Trusted Applications can support both static and shared libraries. In
the latter case, each shared library is identified by a UUID and OP-TEE OS
is in charge of dynamically loading the required shared libraries in the
address space of the Trusted Application when this one uses a resource of
the related library.

In order to support shared library, OP-TEE OS shall be built with
``CFG_ULIBS_SHARED=y``. Shared library binary images are generated as
**.elf** and **.ta** files, like Trusted Applications are, and shall be
installed the same way as Trusted Applications are, see ref:`ta_locations`.

