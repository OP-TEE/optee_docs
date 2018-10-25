.. _cryptographic_implementation:

############################
Cryptographic implementation
############################
This document describes how the TEE Cryptographic Operations API is implemented,
how the default crypto provider may be configured at compile time, and how it
may be replaced by another implementation.

Overview
********
There are several layers from the Trusted Application to the actual crypto
algorithms. Most of the crypto code runs in kernel mode inside the TEE core.
Here is a schematic view of a typical call to the crypto API. The numbers in
square brackets ([1], [2]...) refer to the sections below.

.. code-block:: none

    -   some_function()                             (Trusted App) -
    [1]   TEE_*()                      User space   (libutee.a)
    ------- utee_*() ----------------------------------------------
    [2]       tee_svc_*()              Kernel space
    [3]         crypto_*()                          (libtomcrypt.a and crypto.c)
    [4]           /* LibTomCrypt */                 (libtomcrypt.a)

[1] The TEE Cryptographic Operations API
****************************************
OP-TEE implements the Cryptographic Operations API defined by the GlobalPlatform
association in the :ref:`tee_internal_core_api`. This includes cryptographic
functions that span various cryptographic needs: message digests, symmetric
ciphers, message authentication codes (MAC), authenticated encryption,
asymmetric operations (encryption/decryption or signing/verifying), key
derivation, and random data generation. These functions make up the TEE
Cryptographic Operations API.

The Internal API is implemented in tee_api_operations.c_, which is compiled into
a static library: ``${O}/ta_arm{32,64}-lib/libutee/libutee.a``.

Most API functions perform some parameter checking and manipulations, then
invoke some *utee\_\** function to switch to kernel mode and perform the
low-level work.

The *utee\_\** functions are declared in utee_syscalls.h_ and implemented in
utee_syscalls_asm.S_ They are simple system call wrappers which use the *SVC*
instruction to switch to the appropriate system service in the OP-TEE kernel.

[2] The crypto services
***********************
All cryptography-related system calls are declared in tee_svc_cryp.h_ and
implemented in tee_svc_cryp.c_. In addition to dealing with the usual work
required at the user/kernel interface (checking parameters and copying memory
buffers between user and kernel space), the system calls invoke a private
abstraction layer: the **Crypto API**, which is declared in crypto.h_. It serves
two main purposes:

    1. Allow for alternative implementations, such as hardware-accelerated
       versions.

    2. Provide an easy way to disable some families of algorithms at
       compile-time to save space. See `LibTomCrypt` below.

[3] crypto_*()
**************
The ``crypto_*()`` functions implement the actual algorithms and helper
functions. TEE Core has one global active implementation of this interface. The
default implementation, mostly based on LibTomCrypt_, is as follows:

.. code-block:: c
    :caption: File: core/crypto/crypto.c
    
    /*
     * Default implementation for all functions in crypto.h
     */
    
    #if !defined(_CFG_CRYPTO_WITH_HASH)
    TEE_Result crypto_hash_get_ctx_size(uint32_t algo __unused,
                                        size_t *size __unused)
    {
            return TEE_ERROR_NOT_IMPLEMENTED;
    }
    ...
    #endif /*_CFG_CRYPTO_WITH_HASH*/
    
.. code-block:: c
    :caption: File: core/lib/libtomcrypt/tee_ltc_provider.c
    
    #if defined(_CFG_CRYPTO_WITH_HASH)
    TEE_Result crypto_hash_get_ctx_size(uint32_t algo, size_t *size)
    {
    	/* ... */
    	return TEE_SUCCESS;
    }
    
    #endif /*_CFG_CRYPTO_WITH_HASH*/
    
As shown above, families of algorithms can be disabled and crypto.c_ will
provide default null implementations that will return
``TEE_ERROR_NOT_IMPLEMENTED``.

Public/private key format
*************************
crypto.h_ uses implementation-specific types to hold key data for asymmetric
algorithms. For instance, here is how a public RSA key is represented:

.. code-block:: c
    :caption: File: core/include/crypto/crypto.h

    struct rsa_public_key {
        struct bignum *e;	/* Public exponent */
        struct bignum *n;	/* Modulus */
    };

This is also how such keys are stored inside the TEE object attributes
(``TEE_ATTR_RSA_PUBLIC_KEY`` in this case). ``struct bignum`` is an opaque type,
known to the underlying implementation only. ``struct bignum_ops`` provides
functions so that the system services can manipulate data of this type. This
includes allocation/deallocation, copy, and conversion to or from the big endian
binary format.

.. code-block:: c
    :caption: File: core/include/crypto/crypto.h

    struct bignum *crypto_bignum_allocate(size_t size_bits);

    TEE_Result crypto_bignum_bin2bn(const uint8_t *from, size_t fromsize,
                    struct bignum *to);

    void crypto_bignum_bn2bin(const struct bignum *from, uint8_t *to);
    /*...*/


[4] LibTomCrypt
***************
Some algorithms may be disabled at compile time if they are not needed, in order
to reduce the size of the OP-TEE image and reduces its memory usage. This is
done by setting the appropriate configuration variable. For example:

.. code-block:: bash

    $ make CFG_CRYPTO_AES=n              # disable AES only
    $ make CFG_CRYPTO_{AES,DES}=n        # disable symmetric ciphers
    $ make CFG_CRYPTO_{DSA,RSA,DH,ECC}=n # disable public key algorithms
    $ make CFG_CRYPTO=n                  # disable all algorithms

Please refer to `core/lib/libtomcrypt/sub.mk`_ for the list of all supported
variables.

Note that the application interface is **not** modified when algorithms are
disabled. This means, for instance, that the functions ``TEE_CipherInit()``,
``TEE_CipherUpdate()`` and ``TEE_CipherFinal()`` would remain present in
``libutee.a`` even if all symmetric ciphers are disabled (they would simply
return ``TEE_ERROR_NOT_IMPLEMENTED``).

Add a new crypto implementation
*******************************
To add a new implementation, the default one in `core/lib/libtomcrypt`_ in
combination with what is in `core/crypto`_ should be used as a reference. Here
are the main things to consider when adding a new crypto provider:

    - Put all the new code in its own directory under ``core/lib`` unless it is
      code that will be used regardless of which crypto provider is in use. How
      we are dealing with AES-GCM in `core/crypto`_ could serve as an example.

    - Avoid modifying tee_svc_cryp.c_. It should not be needed.

    - Although not all crypto families need to be defined, all are required for
      compliance to the GlobalPlatform specification.

    - If you intend to make some algorithms optional, please try to re-use the
      same names for configuration variables as the default implementation.

.. Source files
.. _core/crypto: https://github.com/OP-TEE/optee_os/blob/master/core/crypto
.. _crypto.c: https://github.com/OP-TEE/optee_os/blob/master/core/crypto/crypto.c
.. _crypto.h: https://github.com/OP-TEE/optee_os/blob/master/core/include/crypto/crypto.h
.. _core/lib/libtomcrypt: https://github.com/OP-TEE/optee_os/blob/master/core/lib/libtomcrypt
.. _core/lib/libtomcrypt/sub.mk: https://github.com/OP-TEE/optee_os/blob/master/core/lib/libtomcrypt/sub.mk
.. _tee_api_operations.c: https://github.com/OP-TEE/optee_os/blob/master/lib/libutee/tee_api_operations.c
.. _tee_svc_cryp.c: https://github.com/OP-TEE/optee_os/blob/master/core/tee/tee_svc_cryp.c
.. _tee_svc_cryp.h: https://github.com/OP-TEE/optee_os/blob/master/core/include/tee/tee_svc_cryp.h
.. _utee_syscalls.h: https://github.com/OP-TEE/optee_os/blob/master/lib/libutee/include/utee_syscalls.h
.. _utee_syscalls_asm.S: https://github.com/OP-TEE/optee_os/blob/master/lib/libutee/arch/arm/utee_syscalls_asm.S

.. Other links:
.. _LibTomCrypt: https://github.com/libtom/libtomcrypt
