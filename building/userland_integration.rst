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

    # p11 -l --pin 12345 --keypairgen --key-type EC:prime256v1 --label mykey
    Using slot 0 with a present token (0x0)
    Key pair generated:
    Private Key Object; EC
      label:      mykey
      Usage:      sign, derive
      Access:     sensitive, always sensitive, never extractable, local
    Public Key Object; EC  EC_POINT 256 bits
      EC_POINT:   044104e3f89bd32ac8101ba675815fbaf34c4f34bb7bb2d233589983bad934cfa09795d56811747778d22b94e245028d3af6aff9e6abbbdb3a75fe1433182c605868c7
      EC_PARAMS:  06082a8648ce3d030107
      label:      mykey
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
    dynamic_path = /usr/lib/engines-1.1/libpkcs11.so
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

    # OPENSSL_CONF=optee_hsm.conf openssl req -new -engine pkcs11 -keyform engine -key label_mykey -subj "/CN=My CSR" -out mykey_csr.pem
    engine "pkcs11" set.

We can then inspect said CSR:

.. code-block:: none

    $ openssl req -in mykey_csr.pem -text
    Certificate Request:
        Data:
            Version: 1 (0x0)
            Subject: CN = My CSR
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:e3:f8:9b:d3:2a:c8:10:1b:a6:75:81:5f:ba:f3:
                        4c:4f:34:bb:7b:b2:d2:33:58:99:83:ba:d9:34:cf:
                        a0:97:95:d5:68:11:74:77:78:d2:2b:94:e2:45:02:
                        8d:3a:f6:af:f9:e6:ab:bb:db:3a:75:fe:14:33:18:
                        2c:60:58:68:c7
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
            Attributes:
                a0:00
        Signature Algorithm: ecdsa-with-SHA256
             30:45:02:20:61:7e:05:30:cf:4d:d0:93:22:78:9e:45:cf:af:
             3c:83:bb:04:c4:f0:81:f6:9a:5c:97:cd:ac:1e:94:cd:17:1b:
             02:21:00:e7:7f:88:1d:4f:56:b8:e2:87:be:76:de:28:b3:92:
             68:a7:16:3a:56:af:79:2f:98:bd:fd:6d:b3:82:e1:15:6c

Note that the public key matches exactly that which we have previously created
(``04 e3 f8...``). This CSR could then be signed by a CA. For simplicity
purposes, we can also use a self-signed certificate and sign with our own
OP-TEE contained key:

.. code-block:: none

    # OPENSSL_CONF=optee_hsm.conf openssl req -new -engine pkcs11 -keyform engine -key label_mykey -subj "/CN=My CSR" -x509 -out mykey_selfsigned_cert.pem
    engine "pkcs11" set.

Again we can review this self-signed certificate:

.. code-block:: none

    $ openssl x509 -in mykey_selfsigned_cert.pem -text
    Certificate:
        Data:
            Version: 1 (0x0)
            Serial Number:
                3f:8f:c8:c0:de:a8:75:ca:9d:62:79:31:c2:6c:48:f4:fd:50:22:1d
            Signature Algorithm: ecdsa-with-SHA256
            Issuer: CN = My CSR
            Validity
                Not Before: Mar 22 20:19:15 2023 GMT
                Not After : Apr 21 20:19:15 2023 GMT
            Subject: CN = My CSR
            Subject Public Key Info:
                Public Key Algorithm: id-ecPublicKey
                    Public-Key: (256 bit)
                    pub:
                        04:e3:f8:9b:d3:2a:c8:10:1b:a6:75:81:5f:ba:f3:
                        4c:4f:34:bb:7b:b2:d2:33:58:99:83:ba:d9:34:cf:
                        a0:97:95:d5:68:11:74:77:78:d2:2b:94:e2:45:02:
                        8d:3a:f6:af:f9:e6:ab:bb:db:3a:75:fe:14:33:18:
                        2c:60:58:68:c7
                    ASN1 OID: prime256v1
                    NIST CURVE: P-256
        Signature Algorithm: ecdsa-with-SHA256
             30:45:02:20:4a:9d:63:f2:e0:12:4b:46:eb:eb:62:34:9e:86:
             3d:d4:c8:cf:5f:c0:44:fe:8b:71:a0:b8:fa:41:d9:0b:60:3a:
             02:21:00:fb:c2:b3:0a:7b:54:e9:bb:66:7b:8e:f7:11:52:81:
             69:81:a6:cc:d0:bf:a2:7c:f7:2a:67:db:ab:f1:f3:2c:9f
    -----BEGIN CERTIFICATE-----
    MIIBHDCBwwIUP4/IwN6odcqdYnkxwmxI9P1QIh0wCgYIKoZIzj0EAwIwETEPMA0G
    A1UEAwwGTXkgQ1NSMB4XDTIzMDMyMjIwMTkxNVoXDTIzMDQyMTIwMTkxNVowETEP
    MA0GA1UEAwwGTXkgQ1NSMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE4/ib0yrI
    EBumdYFfuvNMTzS7e7LSM1iZg7rZNM+gl5XVaBF0d3jSK5TiRQKNOvav+earu9s6
    df4UMxgsYFhoxzAKBggqhkjOPQQDAgNIADBFAiBKnWPy4BJLRuvrYjSehj3UyM9f
    wET+i3GguPpB2QtgOgIhAPvCswp7VOm7ZnuO9xFSgWmBpszQv6J89ypn26vx8yyf
    -----END CERTIFICATE-----

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

    # OPENSSL_CONF=optee_hsm.conf openssl s_client -engine pkcs11 -connect 192.168.178.34:9876 -cert mykey_selfsigned_cert.pem -keyform engine -key label_mykey
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
