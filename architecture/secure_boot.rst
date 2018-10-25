.. _secure_boot:

###########
Secure boot
###########

Armv8-A - Using the authentication framework in TF-A
****************************************************
This section gives a brief description on how to enable the verification of
OP-TEE using the authentication framework in Trusted Firmware A (TF-A), i.e.,
something that could be used in an Armv8-A environment.

According to user-guide.rst_, there is no additional specific build options for
the verification of OP-TEE. If we have enabled the authentication framework and
specified the ``BL32`` build option when building TF-A, the BL32 related
certificates will be created automatically by the cert_create tool, and then
these certificates will be verified during booting up.

To enable the authentication framework, the following steps should be followed
according to user-guide.rst_. For more details about the authentication
framework, please see auth-framework.rst_ and trusted-board-boot.rst_.

    - Check out a recent version of the `mbed TLS`_ repository and then switch
      to tag mbedtls-2.2.0

    - Besides the normal build options, add the following build options for TF-A

      .. code-block:: bash

        MBEDTLS_DIR=<path of the directory containing mbed TLS sources>
        TRUSTED_BOARD_BOOT=1
        GENERATE_COT=1
        ARM_ROTPK_LOCATION=devel_rsa
        ROT_KEY=<TF-A-PATH/plat/arm/board/common/rotpk/arm_rotprivk_rsa.pem>

Above steps have been tested on FVP platform, all verification steps are OK and
xtest runs successfully without regression.

Armv7-A systems
***************
Unlike for Armv8-A systems where one can use a more standardized way of doing
secure boot by leverage the authentication framework as described above, most
device manufacturers have their own way of doing secure boot. Please reach out
directly to the manufacturer for the device you are working with to be able to
understand how to do secure boot on their devices.

.. _auth-framework.rst : https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/auth-framework.rst
.. _mbed TLS: https://github.com/ARMmbed/mbedtls.git
.. _user-guide.rst: https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/user-guide.rst
.. _trusted-board-boot.rst: https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/trusted-board-boot.rst
