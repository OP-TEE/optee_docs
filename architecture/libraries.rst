.. _libraries:

#########
Libraries
#########

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
Now deprectated, used to the the BigNum library in OP-TEE.

