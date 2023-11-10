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

