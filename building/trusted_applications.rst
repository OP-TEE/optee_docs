.. _build_trusted_applications:

####################
Trusted Applications
####################
This document tells how to implement a Trusted Application for OP-TEE, using
OP-TEE's so called `TA-devkit` to both build and sign the Trusted Application
binary. In this document, a `Trusted Application` running in the OP-TEE os is
referred to as a `TA`. Note that in the default setup a private test key is
distributed along with the :ref:`optee_os` source is used for signing Trusted
Applications. See TASign_ for more details, including offline signing of TAs.

TA Mandatory files
******************
The Makefile for a Trusted Application must be written to rely on OP-TEE
TA-devkit resources in order to successfully build the target application.
TA-devkit is built when one builds :ref:`optee_os`.

.. todo::

    Joakim: We need to add CMake instructions also.

To build a TA, one must provide:

    - **Makefile**, a make file that should set some configuration variables and
      include the TA-devkit make file.

    - **sub.mk**, a make file that lists the sources to build (local source
      files, subdirectories to parse, source file specific build directives).

    - **user_ta_header_defines.h**, a specific ANSI-C header file to define most
      of the TA properties.

    - An implementation of at least the TA entry points, as extern functions:
      ``TA_CreateEntryPoint()``, ``TA_DestroyEntryPoint()``,
      ``TA_OpenSessionEntryPoint()``, ``TA_CloseSessionEntryPoint()``,
      ``TA_InvokeCommandEntryPoint()``

TA file layout example
======================
As an example, :ref:`hello_world` looks like this:

.. code-block:: none

    hello_world/
    ├── ...
    └── ta
        ├── Makefile                  BINARY=<uuid>
        ├── Android.mk                Android way to invoke the Makefile
        ├── sub.mk                    srcs-y += hello_world_ta.c
        ├── include
        │   └── hello_world_ta.h      Header exported to non-secure: TA commands API
        ├── hello_world_ta.c          Implementation of TA entry points
        └── user_ta_header_defines.h  TA_UUID, TA_FLAGS, TA_DATA/STACK_SIZE, ...

TA Makefile Basics
******************
Required variables
==================
The main TA-devkit make file is located in :ref:`optee_os` at
``ta/mk/ta_dev_kit.mk``. The make file supports make targets such as ``all`` and
``clean`` to build a TA or a library and clean the built objects.

The make file expects a couple of configuration variables:

TA_DEV_KIT_DIR
    Base directory of the TA-devkit. Used by the TA-devkit itself to locate its tools.

BINARY and LIBNAME
    These are exclusive, meaning that you cannot use both at the same time. If
    building a TA, ``BINARY`` shall provide the TA filename used to load the TA.
    The built and signed TA binary file will be named ``${BINARY}.ta``. In
    native OP-TEE, it is the TA UUID, used by tee-supplicant to identify TAs. If
    one is building a static library (that will be later linked by a TA), then
    ``LIBNAME`` shall provide the name of the library. The generated library
    binary file will be named ``lib${LIBNAME}.a``

CROSS_COMPILE and CROSS_COMPILE32
    Cross compiler for the TA or the library source files. ``CROSS_COMPILE32``
    is optional. It allows to target AArch32 builds on AArch64 capable systems.
    On AArch32 systems, ``CROSS_COMPILE32`` defaults to ``CROSS_COMPILE``.

Optional variables
==================
Some optional configuration variables can be supported, for example:

O
    Base directory for build objects filetree. If not set, TA-devkit defaults to
    **./out** from the TA source tree base directory.

Example Makefile
================
A typical Makefile for a TA looks something like this

.. code-block:: Makefile

    # Append specific configuration to the C source build (here log=info)
    # The UUID for the Trusted Application
    BINARY=8aaaf200-2450-11e4-abe2-0002a5d5c51b

    # Source the TA-devkit make file
    include $(TA_DEV_KIT_DIR)/mk/ta_dev_kit.mk

.. _build_trusted_applications_submk:

sub.mk directives
=================
The make file expects that current directory contains a file ``sub.mk`` that is
the entry point for listing the source files to build and other specific build
directives. Here are a couple of examples of directives one can implement in a
sub.mk make file:

.. code-block:: Makefile

    # Adds /hello_world_ta.c from current directory to the list of the source
    # file to build and link.
    srcs-y += hello_world_ta.c

    # Includes path **./include/** from the current directory to the include
    # path.
    global-incdirs-y += include/

    # Adds directive -Wno-strict-prototypes only to the file hello_world_ta.c
    cflags-hello_world_ta.c-y += -Wno-strict-prototypes

    # Removes directive -Wno-strict-prototypes from the build directives for
    # hello_world_ta.c only.
    cflags-remove-hello_world_ta.c-y += -Wno-strict-prototypes

    # Adds the static library foo to the list of the linker directive -lfoo.
    libnames += foo

    # Adds the directory path to the libraries pathes list. Archive file
    # libfoo.a is expected in this directory.
    libdirs += path/to/libfoo/install/directory

    # Adds the static library binary to the TA build dependencies.
    libdeps += path/to/greatlib/libgreatlib.a

Android Build Environment
*************************
.. todo::

    Joakim: Move this to the AOSP page?

OP-TEE's TA-devkit supports building in an Android build environment. One can
write an ``Android.mk`` file for the TA (stored side by side with the Makefile).
Android's build system will parse the ``Android.mk`` file for the TA which in
turn will parse a TA-devkit Android make file to locate TA build resources. Then
the Android build will execute a ``make`` command to built the TA through its
generic Makefile file.

A typical ``Android.mk`` file for a TA looks like this (``Android.mk`` for
:ref:`hello_world` is used as an example here).

.. code-block:: Makefile

    # Define base path for the TA sources filetree
    LOCAL_PATH := $(call my-dir)

    # Define the module name as the signed TA binary filename.
    local_module := 8aaaf200-2450-11e4-abe2-0002a5d5c51b.ta

    # Include the devkit Android make script
    include $(OPTEE_OS_DIR)/mk/aosp_optee.mk

TA Mandatory Entry Points
*************************
A TA must implement a couple of mandatory entry points, these are:

.. code-block:: c

    TEE_Result TA_CreateEntryPoint(void)
    {
        /* Allocate some resources, init something, ... */
        ...

        /* Return with a status */
        return TEE_SUCCESS;
    }

    void TA_DestroyEntryPoint(void)
    {
        /* Release resources if required before TA destruction */
        ...
    }

    TEE_Result TA_OpenSessionEntryPoint(uint32_t ptype,
                                        TEE_Param param[4],
                                        void **session_id_ptr)
    {
        /* Check client identity, and alloc/init some session resources if any */
        ...

        /* Return with a status */
        return TEE_SUCCESS;
    }

    void TA_CloseSessionEntryPoint(void *sess_ptr)
    {
        /* check client and handle session resource release, if any */
        ...
    }

    TEE_Result TA_InvokeCommandEntryPoint(void *session_id,
                                          uint32_t command_id,
                                          uint32_t parameters_type,
                                          TEE_Param parameters[4])
    {
        /* Decode the command and process execution of the target service */
        ...

        /* Return with a status */
        return TEE_SUCCESS;
    }

.. _build_ta_properties:

TA Properties
*************
Trusted Application properties shall be defined in a header file named
``user_ta_header_defines.h``, which should contain:

    - ``TA_UUID`` defines the TA uuid value
    - ``TA_FLAGS`` define some of the TA properties
    - ``TA_STACK_SIZE`` defines the RAM size to be reserved for TA stack
    - ``TA_DATA_SIZE`` defines the RAM size to be reserved for TA heap (TEE_Malloc()
      pool)

Refer to :ref:`ta_properties` to understand how to configure these macros.

.. hint::

    UUIDs can be generated using python

    .. code-block:: python

        python -c 'import uuid; print(uuid.uuid4())'

    or in most Linux systems using either

    .. code-block:: bash

        cat /proc/sys/kernel/random/uuid # Linux only
        uuidgen # available from the util-linux package in most distributions



.. _user_ta_header_defines_h:

Example of a property header file
=================================

.. code-block:: c

    #ifndef USER_TA_HEADER_DEFINES_H
    #define USER_TA_HEADER_DEFINES_H

    #define TA_UUID
        { 0x8aaaf200, 0x2450, 0x11e4, \
            { 0xab, 0xe2, 0x00, 0x02, 0xa5, 0xd5, 0xc5, 0x1b} }

    #define TA_FLAGS			(TA_FLAG_EXEC_DDR | \
                            TA_FLAG_SINGLE_INSTANCE | \
                            TA_FLAG_MULTI_SESSION)
    #define TA_STACK_SIZE			(2 * 1024)
    #define TA_DATA_SIZE			(32 * 1024)

    #define TA_CURRENT_TA_EXT_PROPERTIES \
        { "gp.ta.description", USER_TA_PROP_TYPE_STRING, "Foo TA for some purpose." }, \
        { "gp.ta.version", USER_TA_PROP_TYPE_U32, &(const uint32_t){ 0x0100 } }

    #endif /* USER_TA_HEADER_DEFINES_H */

.. note::

    It is recommended to use the ``TA_CURRENT_TA_EXT_PROPERTIES`` as above to
    define extra properties of the TA.

.. note::

    Generating a fresh UUID with suitable formatting for the header file can be
    done using:

    .. code-block:: python

        python -c "import uuid; u=uuid.uuid4(); print(u); \
           n = [', 0x'] * 11; \
           n[::2] = ['{:12x}'.format(u.node)[i:i + 2] for i in range(0, 12, 2)]; \
           print('\n' + '#define TA_UUID\n\t{ ' + \
                 '0x{:08x}'.format(u.time_low) + ', ' + \
                 '0x{:04x}'.format(u.time_mid) + ', ' + \
                 '0x{:04x}'.format(u.time_hi_version) + ', \x5c\n\t\t{ ' + \
                 '0x{:02x}'.format(u.clock_seq_hi_variant) + ', ' + \
                 '0x{:02x}'.format(u.clock_seq_low) + ', ' + \
                 '0x' + ''.join(n) + '} }')"


Checking TA parameters
**********************
GlobalPlatforms TEE Client APIs ``TEEC_InvokeCommand()`` and
``TEE_OpenSession()`` allow clients to invoke a TA with some invocation
parameters: values or references to memory buffers. It is **mandatory** that
TA's verify the parameters types before using the parameters themselves. For
this a TA can rely on the macro ``TEE_PARAM_TYPE_GET(param_type, param_index)``
to get the type of a parameter and check its value according to the expected
parameter.

.. warning::
  Missing verification of expected parameter types may expose arbitrary
  read-and-write exploit primitives within the affected Trusted Application
  (TA), potentially leading to partial control over the TEE in the worst-case
  scenario. For more information, please refer to the GlobalConfusion_ paper,
  which provides detailed insights into this issue.

For example, if a TA expects that command ID 0 comes with ``params[0]`` being a
input value, ``params[1]`` being a output value, and ``params[2]`` being a
in/out memory reference (buffer), then the TA should implemented the following
sequence:

.. code-block:: c

    TEE_Result handle_command_0(void *session, uint32_t cmd_id,
                                uint32_t param_types, TEE_Param params[4])
    {
        uint32_t exp_param_types = TEE_PARAM_TYPES(TEE_PARAM_TYPE_VALUE_INPUT,
                                                   TEE_PARAM_TYPE_VALUE_OUTPUT,
                                                   TEE_PARAM_TYPE_MEMREF_INOUT,
                                                   TEE_PARAM_TYPE_NONE);

        if (param_types != exp_param_types)
            return TEE_ERROR_BAD_PARAMETERS;

        /* process command */
        ...
    }

    TEE_Result TA_InvokeCommandEntryPoint(void *session, uint32_t command_id,
                          uint32_t param_types, TEE_Param params[4])
    {
        switch (command_id) {
        case 0:
            return handle_command_0(session, param_types, params);

        default:
            return TEE_ERROR_NOT_SUPPORTED;
        }
    }

.. warning::
  As stated in the Global API documentation (v1.3.1, section 4.11.1), memory 
  references (``TEE_PARAM_TYPE_MEMREF_*``) underlying buffers are not locked 
  for TEE exclusive access and can therefore be modified by a non-secure client 
  during command processing. It is the reponsability of the TA (or PTA) to ensure 
  data immutability during command execution, if such a property is needed. 
  For example, a TA (or PTA) could copy all input data into a locally allocated buffer
  and only access this copy.

Identifying TA's client
***********************

The GP TEE specification is designed to ensure that TEE sessions are reliable
once created. A TA instance can identify its client login method when a session
is opened. A TA can use the client login credentials to establish or reject
the session. A TA can get its client identity from property
``"gpd.client.identity"`` with the TEE Internal Core API function
``TEE_GetPropertyAsIdentity(()``:

.. code-block:: c

    TEE_Result TA_OpenSessionEntryPoint(uint32_t __unused param_types,
                                        TEE_Param __unused params[4],
                                        void **tee_session)
    {
        TEE_Identity identity = { };
        TEE_Result res = TEE_SUCCESS;

        res = TEE_GetPropertyAsIdentity(TEE_PROPSET_CURRENT_CLIENT,
                                        "gpd.client.identity", &identity);
        if (res)
            return res;

        switch (identity.login) {
        case TEE_LOGIN_PUBLIC:
            return login_public(&identity.uuid, tee_session);
        case TEE_LOGIN_USER:
            return login_user(&identity.uuid, tee_session);
        case TEE_LOGIN_GROUP:
            return login_group(&identity.uuid, tee_session);
        case TEE_LOGIN_REE_KERNEL:
            return login_kernel(&identity.uuid, tee_session);
        case TEE_LOGIN_TRUSTED_APP:
            return login_ta(&identity.uuid, tee_session);
        default:
            return TEE_ERROR_ACCESS_DENIED;
        }
    }

The value of the UUID found in ``identity.uuid`` depends on the login method:

    - When the client is a TA, ``identity.login`` is ``TEE_LOGIN_TRUSTED_APP``
      and ``identity.uuid`` is the client TA UUID;

    - When the non-secure client uses ``TEE_LOGIN_PUBLIC`` or
      ``TEE_LOGIN_REE_KERNEL`` method, the UUID is not used. By convention,
      Linux kernel and U-Boot both set nil UUID (all zeroes).

    - When the non-secure client uses ``TEE_LOGIN_USER`` or ``TEE_LOGIN_GROUP``
      method, the UUID is generated from the UUIDv5 namespace derivation of a
      user ID tag (``"uid=%x"``) or a group ID tag (``"gid=%x"``) in
      ``tee_client_uuid_ns`` namespace (below). The derivation is performed
      by the Linux kernel that verifies that the client's UID/GID is genuine,
      refer to `tee_session_calc_client_uuid()`_.

.. code-block:: c

    static const uuid_t tee_client_uuid_ns = UUID_INIT(0x58ac9ca0, 0x2086, 0x4683,
                                                       0xa1, 0xb8, 0xec, 0x4b,
                                                       0xc0, 0x8e, 0x01, 0xb6);

.. _TASign:

Signing of TAs
**************

All :ref:`REE Filesystem Trusted Applications<ree_fs_ta>` need to be signed. The
signature is verified by :ref:`optee_os` upon loading of the TA. Within the
:ref:`optee_os` source is a directory ``keys``. The public part of
``keys/default_ta.pem`` will be compiled into the :ref:`optee_os` binary and the
signature of each TA will be verified against this key upon loading. Currently
``keys/default_ta.pem`` must contain an RSA key.

.. warning::

    :ref:`optee_os` comes with a default **private** key in its source to
    facilitate easy development, testing, debugging and QA. Never deploy an
    :ref:`optee_os` binary with this key in production. Instead replace this key
    as soon as possible with a public key and keep the private part of the key
    offline, preferably on an HSM.

.. note::

    Currently only a single key for signing TAs is supported by :ref:`optee_os`.

TAs are signed using the ``sign_encrypt.py`` script referenced from
``ta/mk/ta_dev_kit.mk`` in :ref:`optee_os`. Its default behaviour is to sign a
compiled TA binary and attach the signature to form a complete TA for
deployment. For **offline** signing, a three-step process is required: In a
first step a digest of the compiled binary has to be generated, in the second
step this digest is signed offline using the private key and finally in the
third step the binary and its signature are stitched together into the full TA.

Offline Signing of TAs
======================

There are two types of TAs that can be signed offline. The in-tree TAs, which come with the OP-TEE
OS (for example the ``pkcs11`` TA) and are generated during the compilation of the TA DEV KIT. The
second type are any external TAs coming from the user. In both cases however, the signing process
is the same.

Offline signing is done with the following sequence of steps:

0. (Preparation) Generate a 2048 or 4096 bit RSA key for signing in a secure environment and extract 
the public key. For example

.. code-block:: sh

    openssl genrsa -out rsa2048.pem 2048
    openssl rsa -in rsa2048.pem -pubout -out rsa2048_pub.pem 

1. Build the OP-TEE OS with the variable ``TA_PUBLIC_KEY`` set to the public
key generated above

.. code-block:: sh

    TA_PUBLIC_KEY=/path/to/public_key.pem make all

The build script will do two things:

* | It will embed the ``TA_PUBLIC_KEY`` key into the OP-TEE core image, which will be used to
  | authenticate the TAs.

* | It will generate `.stripped.elf` files of the in-tree TAs and sign them with the dummy key
  | pointed to by ``TA_SIGN_KEY``, thus creating `.ta` files. Note that the generated `.ta` files are
  | not to be used as they are not compatible with the public key embedded into the OP-TEE core image.

2. Build any external TA. Same as with the in-tree TAs, the building procedure can use the dummy key 
pointed to by ``TA_SIGN_KEY``, however they are not to be used due to the incompatibility reasons 
mentioned in the paragraph above.

There are now two ways to generate the final ``.ta`` files. Either re-sign the ``.ta`` files with a 
customized ``sign_encrypt.py`` script (left to the user to implement) or stitch the ``.stripped.elf``
files and their signatures together (explained in steps 3-5). 

Export the previously generated custom keypair and the UUID of the TA. In this example the UUID of 
OP-TEE's pkcs11 in-tree TA is used. 

.. code-block:: sh

    export TA_SIGN_KEY=rsa2048.pem
    export TA_PUBLIC_KEY=rsa2048_pub.pem
    export UUID=fd02c9da-306c-48c7-a49c-bbd827ae86ee

3. Manually generate a digest of the generated ``.stripped.elf`` files using

.. code-block:: sh

    sign_encrypt.py digest --key $TA_PUBLIC_KEY --uuid $UUID \
        --elf $UUID.stripped.elf --dig $UUID.dig

.. note::
        It may be necessary to make use of the ``--ta-version`` argument here in some cases, 
        e.g when building Widevine's oemcrypto. Check the make output of optee-os or the 
        particular TAs and see if the version differs.

4. Sign this digest offline, for example with OpenSSL

.. code-block:: sh

    base64 --decode $UUID.dig | \
    openssl pkeyutl -sign -inkey $TA_SIGN_KEY \
        -pkeyopt digest:sha256 -pkeyopt rsa_padding_mode:pss \
        -pkeyopt rsa_pss_saltlen:digest -pkeyopt rsa_mgf1_md:sha256 | \
    base64 > $UUID.sig

or using a Nitrokey HSM (assuming a working OpenSSL configuration for the PKCS11 engine is present)

.. code-block:: sh

    base64 -d $UUID.dig | \
    openssl pkeyutl -engine pkcs11 -keyform engine \
        -sign -inkey "pkcs11:token=<my access token>;type=cert;object=<key label> or id=<key id>" \
        -pkeyopt digest:sha256 -pkeyopt rsa_padding_mode:pss \
        -pkeyopt rsa_pss_saltlen:digest \
        -pkeyopt rsa_mgf1_md:sha256 | \
    base64 > $UUID.sig

When using an HSM, the public key must be extracted and set as ``TA_PUBLIC_KEY``. ``TA_SIGN_KEY`` 
doesn't need to be set in this case, since it is stored in the HSM module.

5. Manually stitch the TA and signature together

.. code-block:: sh

    sign_encrypt.py stitch --key $TA_PUBLIC_KEY --uuid $UUID \
        --elf $UUID.stripped.elf --sig $UUID.sig --out $UUID.ta

.. note::
        If the ``--ta-version`` flag was used in step 3., it needs to be used here as well.

By default, the UUID is taken as the base file name for all files. When signing directly inside 
the optee-os repository the ``$UUID.sig``, ``UUID.dig`` and ``$UUID.ta`` arguments can be omitted.
They were merely provided in this example for completeness. Consult ``sign_encrypt.py --help`` 
for a full list of options and parameters.

.. _tee_session_calc_client_uuid(): https://elixir.bootlin.com/linux/latest/A/ident/tee_session_calc_client_uuid
.. _GlobalConfusion: https://hexhive.epfl.ch/publications/files/24SEC4.pdf
