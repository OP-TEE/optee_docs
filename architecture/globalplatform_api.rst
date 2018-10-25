.. _globalplatform_api:

##################
GlobalPlatform API
##################
Introduction
************
GlobalPlatform_ works across industries to identify, develop and publish
specifications which facilitate the secure and interoperable deployment and
management of multiple embedded applications on secure chip technology. OP-TEE
has support for GlobalPlatform TEE Client API Specification_ v1.0 (GPD_SPE_007)
and TEE Internal Core API Specification v1.1.2 (GPD_SPE_010).


.. _tee_client_api:

TEE Client API
**************
The TEE Client API describes and defines how a client running in a rich
operating environment (REE) should communicate with the TEE. To identify a
Trusted Application (TA) to be used, the client provides an UUID_. All TA's
exposes one or several functions. Those functions corresponds to a so called
``commandID`` which also is sent by the client.

TEE Contexts
============
The TEE Context is used for creating a logical connection between the client and
the TEE. The context must be initialized before the TEE Session can be created.
When the client has completed a job running in secure world, it should finalize
the context and thereby also release resources.

TEE Sessions
============
Sessions are used to create logical connections between a client and a specific
Trusted Application. When the session has been established the client has opened
up the communication channel towards the specified Trusted Application
identified by the ``UUID``. At this stage the client and the Trusted Application
can start to exchange data.


TEE Client API example / usage
==============================
Below you will find the main functions as defined by GlobalPlatform and are used
in the communication between the client and the TEE.

.. code-block:: c

    TEEC_Result TEEC_InitializeContext(
    	const char* name,
    	TEEC_Context* context)

    void TEEC_FinalizeContext(
    	TEEC_Context* context)

    TEEC_Result TEEC_OpenSession (
    	TEEC_Context* context,
    	TEEC_Session* session,
    	const TEEC_UUID* destination,
    	uint32_t connectionMethod,
    	const void* connectionData,
    	TEEC_Operation* operation,
    	uint32_t* returnOrigin)

    void TEEC_CloseSession (
    	TEEC_Session* session)

    TEEC_Result TEEC_InvokeCommand(
    	TEEC_Session* session,
    	uint32_t commandID,
    	TEEC_Operation* operation,
    	uint32_t* returnOrigin)

In principle the commands are called in this order:

.. code-block:: c

    TEEC_InitializeContext(...)
    TEEC_OpenSession(...)
    TEEC_InvokeCommand(...)
    TEEC_CloseSession(...)
    TEEC_FinalizeContext(...)

It is not uncommon that ``TEEC_InvokeCommand(...)`` is called several times in
a row when the session has been established.

For a complete example, please see chapter **5.2 Example 1: Using the TEE Client
API** in the GlobalPlatform TEE Client API Specification_ v1.0.


.. _tee_internal_core_api:

TEE Internal Core API
*********************
The Internal Core API is the API that is exposed to the Trusted Applications
running in the secure world. The TEE Internal API consists of four major parts:

    1. Trusted Storage API for Data and Keys
    2. Cryptographic Operations API
    3. Time API
    4. Arithmetical API

Examples / usage
================
Calling the Internal Core API is done in the same way as described above using
Client API. The best place to find information how this should be done is in the
TEE Internal Core API Specification_ v1.1.2 which contains many examples of how
to call the various APIs. One can also have a look at the examples in the
optee_examples_ git.


.. _extensions:

Extensions
**********
In addition to what is stated in :ref:`tee_internal_core_api`, there are some
non-official extensions in OP-TEE.

Trusted Applications should include header file ``tee_api_defines_extensions.h``
to import the definitions of the extensions. For each extension, a configuration
directive prefixed ``CFG_`` allows one to disable support for the extension when
building the OP-TEE packages.

Cache Maintenance Support
=========================
Following functions have been introduced in order to allow Trusted Applications
to operate with the data cache:

.. code-block:: c

    TEE_Result TEE_CacheClean(char *buf, size_t len);
    TEE_Result TEE_CacheFlush(char *buf, size_t len);
    TEE_Result TEE_CacheInvalidate(char *buf, size_t len);

These functions are available to any Trusted Application defined with the flag
``TA_FLAG_CACHE_MAINTENANCE`` sets on. When not set, each function returns the
error code ``TEE_ERROR_NOT_SUPPORTED``. Within these extensions, a Trusted
Application is able to operate on the data cache, with the following
specification:

.. list-table::
    :widths: 10 60
    :header-rows: 1

    * - Function
      - Description

    * - ``TEE_CacheClean()``
      - Write back to memory any dirty data cache lines. The line is marked as
        not dirty. The valid bit is unchanged.

    * - ``TEE_CacheFlush()``
      - Purges any valid data cache lines. Any dirty cache lines are first
        written back to memory, then the cache line is invalidated.

    * - ``TEE_CacheInvalidate()``
      - Invalidate any valid data cache lines. Any dirty line are not written
        back to memory.

In the following two cases, the error code ``TEE_ERROR_ACCESS_DENIED`` is
returned:

    - The memory range has not the write access, that is
      ``TEE_MEMORY_ACCESS_WRITE`` is not set.
    - The memory is **not** user space memory.


You may disable this extension by setting the following configuration variable
in ``conf.mk``:

.. code-block:: make

    CFG_CACHE_API := n


.. _rsassa_na1:

PKCS#1 v1.5 RSASSA without hash OID
===================================
This extension adds identifer``TEE_ALG_RSASSA_PKCS1_V1_5`` to allow signing and
verifying messages with RSASSA-PKCS1-v1_5, in `RFC 3447`_, without including the
OID of the hash in the signature. You may disable this extension by setting the
following configuration variable in ``conf.mk``:

.. code-block:: make

    CFG_CRYPTO_RSASSA_NA1 := n

The TEE Internal Core API was extended with a new algorithm descriptor.

.. list-table::
    :widths: 10 60
    :header-rows: 1

    * - Algorithm
      - Possible Modes

    * - TEE_ALG_RSASSA_PKCS1_V1_5
      - TEE_MODE_SIGN / TEE_MODE_VERIFY

.. list-table::
    :widths: 10 60
    :header-rows: 1

    * - Algorithm
      - Identifier

    * - TEE_ALG_RSASSA_PKCS1_V1_5
      - 0xF0000830


.. _concat_kdf:

Concat KDF
==========
Support for the Concatenation Key Derivation Function (Concat KDF) according to
`SP 800-56A`_ (*Recommendation for Pair-Wise Key Establishment Schemes Using
Discrete Logarithm Cryptography*) can be found in OP-TEE. You may disable this
extension by setting the following configuration variable in ``conf.mk``:

.. code-block:: make

    CFG_CRYPTO_CONCAT_KDF := n

**Implementation notes**

All key and parameter sizes **must** be multiples of 8 bits. That is:

    - Input parameters: the shared secret (``Z``) and ``OtherInfo``.
    - Output parameter: the derived key (``DerivedKeyingMaterial``).

In addition, the maximum size of the derived key is limited by the size of an
object of type ``TEE_TYPE_GENERIC_SECRET`` (512 bytes). This implementation does
**not** enforce any requirement on the content of the ``OtherInfo`` parameter.
It is the application's responsibility to make sure this parameter is
constructed as specified by the NIST specification if compliance is desired.

**API extension**

To support Concat KDF, the :ref:`tee_internal_core_api` v1.1 was extended with
new algorithm descriptors, new object types, and new object attributes as
described below.

**p.95 Add new object type to TEE_PopulateTransientObject**

The following entry shall be added to **Table 5-8**:

.. list-table::
    :widths: 10 60
    :header-rows: 1

    * - Object type
      - Parts

    * - TEE_TYPE_CONCAT_KDF_Z
      - The ``TEE_ATTR_CONCAT_KDF_Z`` part (input shared secret) must be
        provided.

**p.121 Add new algorithms for TEE_AllocateOperation**

The following entry shall be added to **Table 6-3**:

.. list-table::
    :widths: 10 60
    :header-rows: 1

    * - Algorithm
      - Possible Modes

    * - TEE_ALG_CONCAT_KDF_SHA1_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA224_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA256_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA384_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA512_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA512_DERIVE_KEY
      - TEE_MODE_DERIVE

**p.126 Explain usage of HKDF algorithms in TEE_SetOperationKey**

In the bullet list about operation mode, the following shall be added:

    - For the Concat KDF algorithms, the only supported mode is
      ``TEE_MODE_DERIVE``.

**p.150 Define TEE_DeriveKey input attributes for new algorithms**

The following sentence shall be deleted:

.. code-block:: none

    The TEE_DeriveKey function can only be used with the algorithm
    TEE_ALG_DH_DERIVE_SHARED_SECRET.

The following entry shall be added to **Table 6-7**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Possible operation parameters

    * - TEE_ALG_CONCAT_KDF_SHA1_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA224_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA256_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA384_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA512_DERIVE_KEY
        TEE_ALG_CONCAT_KDF_SHA512_DERIVE_KEY
      - TEE_ATTR_CONCAT_KDF_DKM_LENGTH: up to 512 bytes. This parameter is
        mandatory: TEE_ATTR_CONCAT_KDF_OTHER_INFO

**p.152 Add new algorithm identifiers**

The following entries shall be added to **Table 6-8**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Identifier

    * - TEE_ALG_CONCAT_KDF_SHA1_DERIVE_KEY
      - 0x800020C1

    * - TEE_ALG_CONCAT_KDF_SHA224_DERIVE_KEY
      - 0x800030C1

    * - TEE_ALG_CONCAT_KDF_SHA256_DERIVE_KEY
      - 0x800040C1

    * - TEE_ALG_CONCAT_KDF_SHA384_DERIVE_KEY
      - 0x800050C1

    * - TEE_ALG_CONCAT_KDF_SHA512_DERIVE_KEY
      - 0x800060C1

**p.154 Define new main algorithm**

In **Table 6-9** in section 6.10.1, a new value shall be added to the value
column for row bits ``[7:0]``:

.. list-table::
    :header-rows: 1

    * - Bits
      - Function
      - Value

    * - Bits [7:0]
      - Identifiy the main underlying algorithm itself
      - ...

        0xC1: Concat KDF

The function column for ``bits[15:12]`` shall also be modified to read:

.. list-table::
    :header-rows: 1

    * - Bits
      - Function
      - Value

    * - Bits [15:12]
      - Define the message digest for asymmetric signature algorithms or Concat KDF
      -

**p.155 Add new object type for Concat KDF input shared secret**

The following entry shall be added to **Table 6-10**:

.. list-table::
    :header-rows: 1

    * - Name
      - Identifier
      - Possible sizes

    * - TEE_TYPE_CONCAT_KDF_Z
      - 0xA10000C1
      - 8 to 4096 bits (multiple of 8)

**p.156 Add new operation attributes for Concat KDF**

The following entries shall be added to **Table 6-11**:

.. list-table::
    :header-rows: 1

    * - Name
      - Value
      - Protection
      - Type
      - Comment

    * - TEE_ATTR_CONCAT_KDF_Z
      - 0xC00001C1
      - Protected
      - Ref
      - The shared secret (``Z``)

    * - TEE_ATTR_CONCAT_KDF_OTHER_INFO
      - 0xD00002C1
      - Public
      - Ref
      - ``OtherInfo``

    * - TEE_ATTR_CONCAT_KDF_DKM_LENGTH
      - 0xF00003C1
      - Public
      - Value
      - The length (in bytes) of the derived keying material to be generated,
        maximum 512. This is ``KeyDataLen`` / 8.


.. _hkdf:

HKDF
====
OP-TEE implements the *HMAC-based Extract-and-Expand Key Derivation Function
(HKDF)* as specified in `RFC 5869`_. This file documents the extensions to the
:ref:`tee_internal_core_api` v1.1 that were implemented to support this
algorithm. Trusted Applications should include
``<tee_api_defines_extensions.h>`` to import the definitions.

Note that the implementation follows the recommendations of version 1.1 of the
specification for adding new algorithms. It should make it compatible with
future changes to the official specification. You can disable this extension by
setting the following in ``conf.mk``:

.. code-block:: make

    CFG_CRYPTO_HKDF := n

**p.95 Add new object type to TEE_PopulateTransientObject**

The following entry shall be added to **Table 5-8**:

.. list-table::
    :header-rows: 1

    * - Object type
      - Parts

    * - TEE_TYPE_HKDF_IKM
      - The TEE_ATTR_HKDF_IKM (Input Keying Material) part must be provided.

**p.121 Add new algorithms for TEE_AllocateOperation**

The following entry shall be added to **Table 6-3**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Possible Modes

    * - TEE_ALG_HKDF_MD5_DERIVE_KEY
        TEE_ALG_HKDF_SHA1_DERIVE_KEY
        TEE_ALG_HKDF_SHA224_DERIVE_KEY
        TEE_ALG_HKDF_SHA256_DERIVE_KEY
        TEE_ALG_HKDF_SHA384_DERIVE_KEY
        TEE_ALG_HKDF_SHA512_DERIVE_KEY
        TEE_ALG_HKDF_SHA512_DERIVE_KEY
      - TEE_MODE_DERIVE

**p.126 Explain usage of HKDF algorithms in TEE_SetOperationKey**

In the bullet list about operation mode, the following shall be added:

    - For the HKDF algorithms, the only supported mode is TEE_MODE_DERIVE.

**p.150 Define TEE_DeriveKey input attributes for new algorithms**

The following sentence shall be deleted:

.. code-block:: none

    The TEE_DeriveKey function can only be used with the algorithm
    TEE_ALG_DH_DERIVE_SHARED_SECRET

The following entry shall be added to **Table 6-7**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Possible operation parameters

    * - TEE_ALG_HKDF_MD5_DERIVE_KEY
        TEE_ALG_HKDF_SHA1_DERIVE_KEY
        TEE_ALG_HKDF_SHA224_DERIVE_KEY
        TEE_ALG_HKDF_SHA256_DERIVE_KEY
        TEE_ALG_HKDF_SHA384_DERIVE_KEY
        TEE_ALG_HKDF_SHA512_DERIVE_KEY
        TEE_ALG_HKDF_SHA512_DERIVE_KEY
      - TEE_ATTR_HKDF_OKM_LENGTH: Number of bytes in the Output Keying Material

        TEE_ATTR_HKDF_SALT (optional) Salt to be used during the extract step

        TEE_ATTR_HKDF_INFO (optional) Info to be used during the expand step

**p.152 Add new algorithm identifiers**

The following entries shall be added to **Table 6-8**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Identifier

    * - TEE_ALG_HKDF_MD5_DERIVE_KEY
      - 0x800010C0

    * - TEE_ALG_HKDF_SHA1_DERIVE_KEY
      - 0x800020C0

    * - TEE_ALG_HKDF_SHA224_DERIVE_KEY
      - 0x800030C0

    * - TEE_ALG_HKDF_SHA256_DERIVE_KEY
      - 0x800040C0

    * - TEE_ALG_HKDF_SHA384_DERIVE_KEY
      - 0x800050C0

    * - TEE_ALG_HKDF_SHA512_DERIVE_KEY
      - 0x800060C0

## p.154 Define new main algorithm

In **Table 6-9** in section 6.10.1, a new value shall be added to the value column
for row ``bits [7:0]``:

.. list-table::
    :header-rows: 1

    * - Bits
      - Function
      - Value

    * - Bits [7:0]
      - Identifiy the main underlying algorithm itself
      - ...

        0xC0: HKDF

The function column for ``bits[15:12]`` shall also be modified to read:

.. list-table::
    :header-rows: 1

    * - Bits
      - Function
      - Value

    * - Bits [15:12]
      - Define the message digest for asymmetric signature algorithms or HKDF
      -

**p.155 Add new object type for HKDF input keying material**

The following entry shall be added to **Table 6-10**:

.. list-table::
    :header-rows: 1

    * - Name
      - Identifier
      - Possible sizes

    * - TEE_TYPE_HKDF_IKM
      - 0xA10000C0
      - 8 to 4096 bits (multiple of 8)

**p.156 Add new operation attributes for HKDF salt and info**

The following entries shall be added to **Table 6-11**:

.. list-table::
    :widths: 40 10 10 10 40
    :header-rows: 1

    * - Name
      - Value
      - Protection
      - Type
      - Comment

    * - TEE_ATTR_HKDF_IKM
      - 0xC00001C0
      - Protected
      - Ref
      -

    * - TEE_ATTR_HKDF_SALT
      - 0xD00002C0
      - Public
      - Ref
      -

    * - TEE_ATTR_HKDF_INFO
      - 0xD00003C0
      - Public
      - Ref
      -

    * - TEE_ATTR_HKDF_OKM_LENGTH
      - 0xF00004C0
      - Public
      - Value
      -

.. _pbkdf2:

PBKDF2
======
This document describes the OP-TEE implementation of the key derivation
function, *PBKDF2* as specified in `RFC 2898`_ section 5.2. This RFC is a
republication of PKCS #5 v2.0 from RSA Laboratories' Public-Key Cryptography
Standards (PKCS) series. You may disable this extension by setting the following
configuration variable in ``conf.mk``:

.. code-block:: make

    CFG_CRYPTO_PBKDF2 := n

**API extension**

To support PBKDF2, the :ref:`tee_internal_core_api` v1.1 was extended with a new
algorithm descriptor, new object types, and new object attributes as described
below.

**p.95 Add new object type to TEE_PopulateTransientObject**

The following entry shall be added to **Table 5-8**:

.. list-table::
    :header-rows: 1

    * - Object type
      - Parts

    * - TEE_TYPE_PBKDF2_PASSWORD
      - The TEE_ATTR_PBKDF2_PASSWORD part must be provided.

**p.121 Add new algorithms for TEE_AllocateOperation**

The following entry shall be added to **Table 6-3**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Possible Modes

    * - TEE_ALG_PBKDF2_HMAC_SHA1_DERIVE_KEY
      - TEE_MODE_DERIVE

**p.126 Explain usage of PBKDF2 algorithm in TEE_SetOperationKey**

In the bullet list about operation mode, the following shall be added:

    - For the PBKDF2 algorithm, the only supported mode is TEE_MODE_DERIVE.

**p.150 Define TEE_DeriveKey input attributes for new algorithms**

The following sentence shall be deleted:

.. code-block:: none

    The TEE_DeriveKey function can only be used with the algorithm
    TEE_ALG_DH_DERIVE_SHARED_SECRET

The following entry shall be added to **Table 6-7**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Possible operation parameters

    * - TEE_ALG_PBKDF2_HMAC_SHA1_DERIVE_KEY
      - TEE_ATTR_PBKDF2_DKM_LENGTH: up to 512 bytes. This parameter is
        mandatory.

        TEE_ATTR_PBKDF2_SALT

        TEE_ATTR_PBKDF2_ITERATION_COUNT: This parameter is mandatory.

**p.152 Add new algorithm identifiers**

The following entries shall be added to **Table 6-8**:

.. list-table::
    :header-rows: 1

    * - Algorithm
      - Identifier

    * - TEE_ALG_PBKDF2_HMAC_SHA1_DERIVE_KEY
      - 0x800020C2

**p.154 Define new main algorithm**

In **Table 6-9** in section 6.10.1, a new value shall be added to the value
column for row ``bits [7:0]``:

.. list-table::
    :header-rows: 1

    * - Bits
      - Function
      - Value

    * - Bits [7:0]
      - Identifiy the main underlying algorithm itself
      - ...

        0xC2: PBKDF2

The function column for ``bits[15:12]`` shall also be modified to read:

.. list-table::
    :header-rows: 1

    * - Bits
      - Function
      - Value

    * - Bits [15:12]
      - Define the message digest for asymmetric signature algorithms or PBKDF2
      -

**p.155 Add new object type for PBKDF2 password**

The following entry shall be added to **Table 6-10**:

.. list-table::
    :header-rows: 1

    * - Name
      - Identifier
      - Possible sizes

    * - TEE_TYPE_PBKDF2_PASSWORD
      - 0xA10000C2
      - 8 to 4096 bits (multiple of 8)

**p.156 Add new operation attributes for Concat KDF**

The following entries shall be added to **Table 6-11**:

.. list-table::
    :widths: 40 10 10 10 40
    :header-rows: 1

    * - Name
      - Value
      - Protection
      - Type
      - Comment

    * - TEE_ATTR_PBKDF2_PASSWORD
      - 0xC00001C2
      - Protected
      - Ref
      -

    * - TEE_ATTR_PBKDF2_SALT
      - 0xD00002C2
      - Public
      - Ref
      -

    * - TEE_ATTR_PBKDF2_ITERATION_COUNT
      - 0xF00003C2
      - Public
      - Value
      -

    * - TEE_ATTR_PBKDF2_DKM_LENGTH
      - 0xF00004C2
      - Public
      - Value
      - The length (in bytes) of the derived keying material to be generated,
        maximum 512.


.. _GlobalPlatform: https://globalplatform.org
.. _optee_examples: https://github.com/linaro-swg/optee_examples
.. _TZC-400: http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0504c/index.html
.. _RFC 2898: https://www.ietf.org/rfc/rfc2898.txt
.. _RFC 3447: https://tools.ietf.org/html/rfc3447#section-8.2
.. _RFC 5869: https://tools.ietf.org/html/rfc5869
.. _Specification: https://globalplatform.org/specs-library/?filter-committee=tee
.. _SP 800-56A: http://csrc.nist.gov/publications/nistpubs/800-56A/SP800-56A_Revision1_Mar08-2007.pdf
.. _UUID: https://en.wikipedia.org/wiki/Universally_unique_identifier
