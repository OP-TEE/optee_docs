.. _userland_integration:

##########################
Linux userland integration
##########################

This document gives pointers on how particular features of OP-TEE may be used
from the Linux userland in typical application scenarios.

PKCS#11 driver
**************

A common use-case is the integration of OP-TEE to securely store asymmetric
keys inside the secure enclave. For example, when using TLS with client
certificates, the corresponding private keys would reside securely within
OP-TEE. If this client certificate is then used from within userspace, the
corresponding cryptographic primitives are relayed to OP-TEE which establishes
the connection using the requested client certificate on behalf of the
application. However, the key itself never leaves secure storage (this is where
it is created and resides).

The way this is done is via PKCS#11 (aka Cryptoki API). PKCS#11 specifies a
number of standard calls to relay cryptographic requests (such as a signing
operation) to a third party module.  Such a module may be a smart card or, in
the case of OP-TEE, it is a software PKCS#11 trusted application that appears
to the userland as one. This trusted application is accessed using a shared
object (dynamic library) which serves as the "glue" to translate cryptographic
requests into OP-TEE calls. This shared object is ``libckteec.so`` which is
part of the `OP-TEE client tools <https://github.com/OP-TEE/optee_client>`_.

Once OP-TEE has been compiled with the PKCS#11 TA, the client tools shared
object has been built and the OP-TEE supplicant has been started, we can use
``pkcs11-tool`` of the `OpenSC project <https://github.com/OpenSC/OpenSC>`_ to
initiate first communication with the emulated smart card.  In the following,
we assume that ``libckteec.so`` has been installed in
``/usr/lib/libckteec.so``. For simplicity reasons, we define an alias to call
pkcs11-tool using the appropriate PKCS#11 module.

.. code-block:: none

    # alias p11="pkcs11-tool --module /usr/lib/libckteec.so"
    # p11 --show-info
    Cryptoki version 2.40
    Manufacturer     Linaro
    Library          OP-TEE PKCS11 Cryptoki library (ver 0.1)
    Using slot 0 with a present token (0x0)

.. hint::

   When testing OP-TEE under QEMU, OpenSC should be built by default as well
   and the ``pkcs11-tool`` should be available without modifications to the
   configuration. It can be explicitly requested by using the

   .. code-block:: bash

       make BR2_PACKAGE_OPENSC=y

   build parameter when compiling OP-TEE.

This tells us the library code is already working. We can now display the
different "slots". You can think of them as different "card readers" for
virtual smart cards. In a typical use case, only one slot is used for a single
smart card.

.. code-block:: none

    # p11 --list-slots
    Available slots:
    Slot 0 (0x0): OP-TEE PKCS11 TA - TEE UUID 94e9ab89-4c43-56ea-8b35-45dc07226830
      token state:   uninitialized
    Slot 1 (0x1): OP-TEE PKCS11 TA - TEE UUID 94e9ab89-4c43-56ea-8b35-45dc07226830
      token state:   uninitialized
    Slot 2 (0x2): OP-TEE PKCS11 TA - TEE UUID 94e9ab89-4c43-56ea-8b35-45dc07226830
      token state:   uninitialized

Observe that the connection to the TA is also successfully working and it is
showing three inserted (but "empty", uninitialized) smart cards/tokens. Before
we are able to create keys on these tokens, we need to initialize them with a
SO-PIN and PIN.  The SO-PIN is the "super pin", while the PIN is the "user
pin". The concept is likely familiar to you from the SIM card of your phone,
where the PUK acts as the "super pin".

First, we initialize the SO-PIN of slot 0 and name our token "mytoken":

.. code-block:: none

    # p11 --init-token --label mytoken --so-pin 1234567890
    Using slot 0 with a present token (0x0)
    Token successfully initialized

We have successfully initialized the SO-PIN to "1234567890". Now we "log in"
into the token using that SO-PIN and, using the SO-PIN authorization,
initialize the PIN of the token to "12345":

.. code-block:: none

    # p11 --label mytoken --login --so-pin 1234567890 --init-pin --pin 12345
    Using slot 0 with a present token (0x0)
    User PIN successfully initialized

We can now verify that the token has been successfully initialized:

.. code-block:: none

    # p11 --list-slots
    Available slots:
    Slot 0 (0x0): OP-TEE PKCS11 TA - TEE UUID 94e9ab89-4c43-56ea-8b35-45dc07226830
      token label        : mytoken
      token manufacturer : Linaro
      token model        : OP-TEE TA
      token flags        : login required, rng, token initialized, PIN initialized
      hardware version   : 0.0
      firmware version   : 0.1
      serial num         : 0000000000000000
      pin min/max        : 4/128
    Slot 1 (0x1): OP-TEE PKCS11 TA - TEE UUID 94e9ab89-4c43-56ea-8b35-45dc07226830
      token state:   uninitialized
    Slot 2 (0x2): OP-TEE PKCS11 TA - TEE UUID 94e9ab89-4c43-56ea-8b35-45dc07226830
      token state:   uninitialized

Now we have a fully initialized token but it still contains no keys. To list
what cryptographic primitives the particular OP-TEE version offers, you can
query the supported mechanisms:

.. code-block:: none

    # p11 --list-mechanisms
    Using slot 0 with a present token (0x0)
    Supported mechanisms:
      SHA224-RSA-PKCS-PSS, keySize={256,4096}, sign, verify
      SHA224-RSA-PKCS, keySize={256,4096}, sign, verify
      SHA512-RSA-PKCS-PSS, keySize={256,4096}, sign, verify
      SHA384-RSA-PKCS-PSS, keySize={256,4096}, sign, verify
      SHA256-RSA-PKCS-PSS, keySize={256,4096}, sign, verify
      SHA512-RSA-PKCS, keySize={256,4096}, sign, verify
      SHA384-RSA-PKCS, keySize={256,4096}, sign, verify
      SHA256-RSA-PKCS, keySize={256,4096}, sign, verify
      SHA1-RSA-PKCS-PSS, keySize={256,4096}, sign, verify
      RSA-PKCS-OAEP, keySize={256,4096}, encrypt, decrypt
      SHA1-RSA-PKCS, keySize={256,4096}, sign, verify
      MD5-RSA-PKCS, keySize={256,4096}, sign, verify
      RSA-PKCS-PSS, sign, verify
      RSA-PKCS, keySize={256,4096}, encrypt, decrypt, sign, verify
      RSA-PKCS-KEY-PAIR-GEN, keySize={256,4096}, generate_key_pair
      ECDSA-SHA512, keySize={160,521}, sign, verify
      ECDSA-SHA384, keySize={160,521}, sign, verify
      ECDSA-SHA256, keySize={160,521}, sign, verify
      ECDSA-SHA224, keySize={160,521}, sign, verify
      ECDSA-SHA1, keySize={160,521}, sign, verify
      ECDSA, keySize={160,521}, sign, verify
      ECDSA-KEY-PAIR-GEN, keySize={160,521}, generate_key_pair
      mechtype-0x272, keySize={32,128}, sign, verify
      mechtype-0x262, keySize={32,128}, sign, verify
      mechtype-0x252, keySize={24,128}, sign, verify
      mechtype-0x257, keySize={14,64}, sign, verify
      SHA-1-HMAC-GENERAL, keySize={10,64}, sign, verify
      MD5-HMAC-GENERAL, keySize={8,64}, sign, verify
      SHA512-HMAC, keySize={32,128}, sign, verify
      SHA384-HMAC, keySize={32,128}, sign, verify
      SHA256-HMAC, keySize={24,128}, sign, verify
      SHA224-HMAC, keySize={14,64}, sign, verify
      SHA-1-HMAC, keySize={10,64}, sign, verify
      MD5-HMAC, keySize={8,64}, sign, verify
      SHA512, digest
      SHA384, digest
      SHA256, digest
      SHA224, digest
      SHA-1, digest
      MD5, digest
      GENERIC-SECRET-KEY-GEN, keySize={1,4096}, generate
      AES-KEY-GEN, keySize={16,32}, generate
      AES-CBC-ENCRYPT-DATA, derive
      AES-ECB-ENCRYPT-DATA, derive
      mechtype-0x108B, keySize={16,32}, sign, verify
      AES-CMAC, keySize={16,32}, sign, verify
      mechtype-0x1089, keySize={16,32}, encrypt, decrypt
      AES-CTR, keySize={16,32}, encrypt, decrypt
      AES-CBC-PAD, keySize={16,32}, encrypt, decrypt
      AES-CBC, keySize={16,32}, encrypt, decrypt, wrap, unwrap
      AES-ECB, keySize={16,32}, encrypt, decrypt, wrap, unwrap

In our case, we would want to create an elliptic curve keypair on P-256 (aka
secp256r1 or prime256v1). As you can see, this is supported ("ECDSA-KEY-PAIR-GEN" supports
between 160 and 521 bit curves).

.. code-block:: none

    # p11 -l --pin 12345 --keypairgen --key-type EC:secp521r1 --label mykey --id 1234
    Using slot 0 with a present token (0x0)
    Key pair generated:
    Private Key Object; EC
      label:      mykey
      ID:         1234
      Usage:      sign, derive
      Access:     sensitive, always sensitive, never extractable, local
    Public Key Object; EC  EC_POINT 528 bits
        EC_POINT:   04818504004d1f3a6022557f1c4ef7403679e453e2cecc13e0c1c8ae4e39098232918f875c54d87ca9142a6f7309beb69c6208d58a4aabde024f5b2b34b881fe68f8749717480061e49dc9034ae13a478131ad5f90d750334a7e02ec47172f2e087d6289ff780cb4e7992fa0e5ae098f2ab246b13b9e5ab376043661603e3a1febd3fb8c0baadcf2
        EC_PARAMS:  06052b81040023 (OID 1.3.132.0.35)
      label:      mykey
      ID:         1234
      Usage:      verify, derive
      Access:     local

You can see the public key, which is a point on the elliptic curve. The byte
``04`` at byte offset 2 indicates that this point is represented in
uncompressed affine representation, i.e., X and Y coordinates follow that byte
directly. This format is not ideal to interface common libraries, however.
Especially when using PKI with X.509 certificates, we typically want a
PEM-formatted CSR to be able to create a certificate from.

For this, we create a small configuration file for OpenSSL and call it
``optee_hsm.conf``. It references a library of `libp11
<https://github.com/OpenSC/libp11>`_ which acts as a driver that enables
OpenSSL to interface with a PKCS#11 library.

.. code-block::

    openssl_conf = openssl_conf

    [openssl_conf]
    engines = engine_section

    [engine_section]
    pkcs11 = pkcs11_section

    [pkcs11_section]
    engine_id = pkcs11
    dynamic_path = /usr/lib/engines-3/pkcs11.so
    MODULE_PATH = /usr/lib/libckteec.so
    PIN = 12345

    [req]
    distinguished_name = req_distinguished_name

    [req_distinguished_name]

.. hint::

   When testing OP-TEE under QEMU, libp11 is not compiled by default. For easy
   access to this library, you can build OP-TEE using the command

   .. code-block:: none

       make BR2_PACKAGE_OPENSC=y BR2_PACKAGE_LIBOPENSSL=y BR2_PACKAGE_LIBOPENSSL_BIN=y BR2_PACKAGE_LIBP11=y

   This will ensure that OpenSC (for the command line utility ``pkcs11-tool``),
   OpenSSL, and libp11 are all built and installed in the QEMU environment.
   Note that in that environment, ``libpkcs11.so`` will reside at
   ``/usr/lib/engines-1.1/libpkcs11.so``.

Then, we can ask OpenSSL to create a CSR from the key we have previously created:

.. code-block:: none

    # OPENSSL_CONF=optee_hsm.conf openssl req -new -engine pkcs11 -keyform engine -key 1234 -subj "/CN=My CSR" -out mykey_csr.pem
    engine "pkcs11" set.

We can then inspect said CSR:

.. code-block:: none

    $ openssl req -in mykey_csr.pem -text -noout
    Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            25:00:b4:69:fd:e8:b2:7b:21:73:d7:84:87:6b:60:ff:cc:50:ea:ca
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN = My CSR
        Validity
            Not Before: Sep 23 10:28:02 2024 GMT
            Not After : Oct 23 10:28:02 2024 GMT
        Subject: CN = My CSR
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (521 bit)
                pub:
                    04:00:4d:1f:3a:60:22:55:7f:1c:4e:f7:40:36:79:
                    e4:53:e2:ce:cc:13:e0:c1:c8:ae:4e:39:09:82:32:
                    91:8f:87:5c:54:d8:7c:a9:14:2a:6f:73:09:be:b6:
                    9c:62:08:d5:8a:4a:ab:de:02:4f:5b:2b:34:b8:81:
                    fe:68:f8:74:97:17:48:00:61:e4:9d:c9:03:4a:e1:
                    3a:47:81:31:ad:5f:90:d7:50:33:4a:7e:02:ec:47:
                    17:2f:2e:08:7d:62:89:ff:78:0c:b4:e7:99:2f:a0:
                    e5:ae:09:8f:2a:b2:46:b1:3b:9e:5a:b3:76:04:36:
                    61:60:3e:3a:1f:eb:d3:fb:8c:0b:aa:dc:f2
                ASN1 OID: secp521r1
                NIST CURVE: P-521
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                1A:B7:81:9A:4E:F8:23:AD:6C:B8:E2:EF:65:F6:73:11:39:93:82:66
    Signature Algorithm: ecdsa-with-SHA256
    Signature Value:
        30:81:87:02:42:01:cf:a4:88:06:4e:96:13:13:39:a9:ac:cb:
        29:4d:bb:16:f2:6a:05:ec:8d:cf:3f:f1:39:84:6e:a5:d0:b2:
        e6:6a:d8:81:c6:7b:32:29:8f:75:72:05:22:9d:20:4b:f7:bf:
        7b:fb:23:33:d5:97:9c:83:7c:d7:df:1a:83:c8:5b:ab:e1:02:
        41:0f:f6:ee:fe:46:09:e6:49:7d:2c:3d:df:0b:45:e3:35:b4:
        8e:8d:ce:25:0f:22:3d:6d:36:6b:cb:3a:b6:ab:c2:d3:e6:32:
        1b:83:ab:9e:fe:82:f1:f6:23:ce:20:58:39:39:c2:de:ac:7a:
        e7:71:4a:09:91:4d:0a:f0:51:15:77:b5

Note that the public key matches exactly that which we have previously created
(``04 e3 f8...``). This CSR could then be signed by a CA. For simplicity
purposes, we can also use a self-signed certificate and sign with our own
OP-TEE contained key:

.. code-block:: none

    # OPENSSL_CONF=optee_hsm.conf openssl req -new -engine pkcs11 -keyform engine -key 1234 -subj "/CN=My CSR" -x509 -out mykey_selfsigned_cert.pem
    engine "pkcs11" set.

Again we can review this self-signed certificate:

.. code-block:: none

    $ openssl x509 -in mykey_selfsigned_cert.pem -text -noout
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                25:00:b4:69:fd:e8:b2:7b:21:73:d7:84:87:6b:60:ff:cc:50:ea:ca
            Signature Algorithm: ecdsa-with-SHA256
            Issuer: CN = My CSR
            Validity
                Not Before: Sep 23 10:28:02 2024 GMT
                Not After : Oct 23 10:28:02 2024 GMT
            Subject: CN = My CSR
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (521 bit)
                    pub:
                        04:00:4d:1f:3a:60:22:55:7f:1c:4e:f7:40:36:79:
                        e4:53:e2:ce:cc:13:e0:c1:c8:ae:4e:39:09:82:32:
                        91:8f:87:5c:54:d8:7c:a9:14:2a:6f:73:09:be:b6:
                        9c:62:08:d5:8a:4a:ab:de:02:4f:5b:2b:34:b8:81:
                        fe:68:f8:74:97:17:48:00:61:e4:9d:c9:03:4a:e1:
                        3a:47:81:31:ad:5f:90:d7:50:33:4a:7e:02:ec:47:
                        17:2f:2e:08:7d:62:89:ff:78:0c:b4:e7:99:2f:a0:
                        e5:ae:09:8f:2a:b2:46:b1:3b:9e:5a:b3:76:04:36:
                        61:60:3e:3a:1f:eb:d3:fb:8c:0b:aa:dc:f2
                    ASN1 OID: secp521r1
                    NIST CURVE: P-521
            X509v3 extensions:
                X509v3 Subject Key Identifier: 
                    1A:B7:81:9A:4E:F8:23:AD:6C:B8:E2:EF:65:F6:73:11:39:93:82:66
        Signature Algorithm: ecdsa-with-SHA256
        Signature Value:
            30:81:87:02:42:01:cf:a4:88:06:4e:96:13:13:39:a9:ac:cb:
            29:4d:bb:16:f2:6a:05:ec:8d:cf:3f:f1:39:84:6e:a5:d0:b2:
            e6:6a:d8:81:c6:7b:32:29:8f:75:72:05:22:9d:20:4b:f7:bf:
            7b:fb:23:33:d5:97:9c:83:7c:d7:df:1a:83:c8:5b:ab:e1:02:
            41:0f:f6:ee:fe:46:09:e6:49:7d:2c:3d:df:0b:45:e3:35:b4:
            8e:8d:ce:25:0f:22:3d:6d:36:6b:cb:3a:b6:ab:c2:d3:e6:32:
            1b:83:ab:9e:fe:82:f1:f6:23:ce:20:58:39:39:c2:de:ac:7a:
            e7:71:4a:09:91:4d:0a:f0:51:15:77:b5

To test our self-signed certificate as a client certificate, we first need to
initialize a TLS server. This can either be done on a remote machine or
locally. For the server we will again use a self-signed certificate (but simply
store the corresponding private key in a file).

.. code-block:: none

    $ openssl ecparam -genkey -name prime256v1 -out server_key.pem
    $ openssl req -new -x509 -key server_key.pem -subj '/CN=Server' -out server_cert.pem
    $ openssl s_server -accept 9876 -cert server_cert.pem -key server_key.pem -www -Verify 1
    verify depth is 1, must return a certificate
    Using default temp DH parameters
    ACCEPT

This starts a HTTPS server which listens at port 9876 and requires a TLS client
certificate. We can validate that the connection to the server is refused if no
client certificate is provided. Assume that ``192.168.178.34`` is the IPv4
address of the server:

.. code-block:: none

    $ curl -k https://192.168.178.34:9876
    curl: (56) OpenSSL SSL_read: error:0A00045C:SSL routines::tlsv13 alert certificate required, errno 0

Now on our OP-TEE device we can use OpenSSL to establish a connection using our
OP-TEE stored client certificate:

.. code-block:: none

    # OPENSSL_CONF=optee_hsm.conf openssl s_client -engine pkcs11 -connect 192.168.178.34:9876 -cert mykey_selfsigned_cert.pem -keyform engine -key 1234
    engine "pkcs11" set.
    CONNECTED(00000004)
    Can't use SSL_get_servername
    depth=0 CN = Server
    verify error:num=18:self signed certificate
    verify return:1
    depth=0 CN = Server
    verify return:1
    ---
    Certificate chain
     0 s:CN = Server
       i:CN = Server
    ---
    Server certificate
    -----BEGIN CERTIFICATE-----
    MIIBeDCCAR2gAwIBAgIUDnUzOcNS9AgeJhvVmp73wF5DwxQwCgYIKoZIzj0EAwIw
    ETEPMA0GA1UEAwwGU2VydmVyMB4XDTIzMDMyMjIwMjMwMloXDTIzMDQyMTIwMjMw
    MlowETEPMA0GA1UEAwwGU2VydmVyMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE
    qnCvLjLa1XWBtY1OQjaHa60re5vnZ2WY555XSsFCe2RoF7wGBDDrdXKkQz9Vy0t4
    d5OC6VMcFhia967nGa5zPqNTMFEwHQYDVR0OBBYEFBKTMLG057a/a2exmeF7dHVH
    85D0MB8GA1UdIwQYMBaAFBKTMLG057a/a2exmeF7dHVH85D0MA8GA1UdEwEB/wQF
    MAMBAf8wCgYIKoZIzj0EAwIDSQAwRgIhAIeKwlghSkhA8zvpXsl9y6WSCXo9fRzt
    DSl6myUsgac/AiEAhipKSjVQAvJAqXIecmMylqjY79XVzrbxKWYjsL1XdLw=
    -----END CERTIFICATE-----
    subject=CN = Server

    issuer=CN = Server

    ---
    No client certificate CA names sent
    Requested Signature Algorithms: ECDSA+SHA256:ECDSA+SHA384:ECDSA+SHA512:Ed25519:Ed448:RSA-PSS+SHA256:RSA-PSS+SHA384:RSA-PSS+SHA512:RSA-PSS+SHA256:RSA-PSS+SHA384:RSA-PSS+SHA512:RSA+SHA256:RSA+SHA384:RSA+SHA512:ECDSA+SHA224:RSA+SHA224
    Shared Requested Signature Algorithms: ECDSA+SHA256:ECDSA+SHA384:ECDSA+SHA512:Ed25519:Ed448:RSA-PSS+SHA256:RSA-PSS+SHA384:RSA-PSS+SHA512:RSA-PSS+SHA256:RSA-PSS+SHA384:RSA-PSS+SHA512:RSA+SHA256:RSA+SHA384:RSA+SHA512
    Peer signing digest: SHA256
    Peer signature type: ECDSA
    Server Temp Key: X25519, 253 bits
    ---
    SSL handshake has read 817 bytes and written 797 bytes
    Verification error: self signed certificate
    ---
    New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
    Server public key is 256 bit
    Secure Renegotiation IS NOT supported
    Compression: NONE
    Expansion: NONE
    No ALPN negotiated
    Early data was not sent
    Verify return code: 18 (self signed certificate)
    [...]

When connected, you can type "GET /" and press return to get a HTML response
back from the HTTPS server, which will echo your client certificate inside a
HTML page.
