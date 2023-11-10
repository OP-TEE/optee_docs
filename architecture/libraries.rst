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

.. _libutee:

libutee
*******
The :ref:`tee_internal_core_api` describes services that are provided to Trusted
Applications. **libutee** is a library that implements this API.

libutee is a static library the Trusted Applications shall statically link
against. Trusted Applications do execute in non-privileged secure userspace and
libutee also aims at being executed in the non-privileged secure userspace.

Some services for this API are fully statically implemented inside the libutee
library while some services for the API are implemented inside the OP-TEE core
(privileged level) and libutee calls such services through system calls.

.. _libmpa:

libmpa
******
Now deprecated, used to provide the BigNum library in OP-TEE.

