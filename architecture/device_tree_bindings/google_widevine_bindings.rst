.. _google_widevine_bindings:

####################################
Google Widevine device-tree bindings
####################################

.. code-block:: yaml

  %YAML 1.2
  ---
  $id: http://devicetree.org/schemas/options/op-tee/google,widevine.yaml#
  $schema: http://devicetree.org/meta-schemas/core.yaml#

  title: Google Widevine initialization parameters

  maintainers:
    - Jeffrey Kardatzke <jkardatzke@chromium.org>
    - Yi Chou <yich@chromium.org>

  description:
    Widevine is Google's content protection system for DRM (digital rights
    management) contents.
    The necessary fields to initialize the Widevine related functions in
    OP-TEE. This node does not represent a real device, but serves as a
    place for passing data between firmware and OP-TEE.
    The content of this node should not be shared with the Linux kernel.

  properties:
    op-tee,hardware-unique-key:
      $ref: /schemas/types.yaml#/definitions/uint8-array
      maxItems: 32
      description: |
        The hardware-unique key of the OP-TEE. It will be used to derive
        the secure storage key.
        For more information, please reference:
        https://optee.readthedocs.io/en/latest/architecture/porting_guidelines.html#hardware-unique-key

    tcg,tpm-auth-public-key:
      $ref: /schemas/types.yaml#/definitions/uint8-array
      maxItems: 1024
      description: |
        The TPM auth public key. Used to communicate the TPM from OP-TEE.
        The format of data should be TPM2B_PUBLIC.
        For more information, please reference the 12.2.5 section:
        https://trustedcomputinggroup.org/wp-content/uploads/TCG_TPM2_r1p59_Part2_Structures_pub.pdf

    google,widevine-root-of-trust-ecc-p256:
      $ref: /schemas/types.yaml#/definitions/uint8-array
      maxItems: 32
      description: |
        The Widevine root of trust secret. Used to sign the Widevine
        request in OP-TEE. The value is an ECC NIST P-256 scalar.
        For more information, please reference the G.1.2 section:
        https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-186.pdf

  required:
    - op-tee,hardware-unique-key
    - tcg,tpm-auth-public-key
    - google,widevine-root-of-trust-ecc-p256

  additionalProperties: false

  examples:
    - |
      options {
        google,widevine {
          op-tee,hardware-unique-key = [
            12 f7 98 d2 0e d2 85 92 a5 82 bf 98 b8 99 2b c0
            c6 6f 19 85 79 86 65 18 55 eb ff 9b 6c c0 ac 27
          ];
          tcg,tpm-auth-public-key = [
            00 76 00 23 00 0b 00 02 04 b2 00 20 e1 47 bf 27
            e1 74 30 c8 16 ab 72 4d 5c 77 e1 5c 61 2d 56 81
            b3 35 cd 9d eb 67 41 37 69 f0 32 41 00 10 00 10
            00 03 00 10 00 20 70 9a df 50 f9 0f d5 f4 40 e0
            ea 2c e8 f2 26 9f 0e 5c 02 70 16 c3 6c c1 83 03
            2d 04 10 bd 85 7a 00 20 83 03 c2 66 6e 01 32 34
            5c 5e 80 22 c7 48 24 3c 70 6b b8 e4 24 42 74 a9
            cf fc ab f8 30 e9 de 51
          ];
          google,widevine-root-of-trust-ecc-p256 = [
            ac 0d 86 c3 d7 b5 b7 a2 6f c3 d9 93 f7 de bc bb
            d5 c4 25 9b 21 5f 36 af b5 dd 6d 29 9d 08 c0 10
          ];
        };
      };
