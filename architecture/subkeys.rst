.. _subkeys:

#######
Subkeys
#######
Subkeys is an OP-TEE-specific implementation to provide a public key 
hierarchy. Subkeys can be delegated to allow different actors to sign
different TAs without sharing a private key.

The first key in the chain is verified using a root key. Example Subkey
hierarchy:


.. uml::
	:width: 800

	object "Root key" as root
	object "First-level subkey" as first
	object "Second-level subkey" as second

	object root {
		PublicKey
	}

	object first {
		Signature
		UUID
		PublicKey
	}
	
	object second {
		Signature
		UUID
		PublicKey
	}

	root -> first
	first -> second

Each subkey defines a UUIDv5 namespace [#f1]_ to which another signed
subkey or TA must belong. This avoids that one subkey can be used to sign
TAs which should be signed with a different key. Since the UUIDs are
restricted by this there is also a special kind of subkey, called identity
subkey, which uses the same UUID as the TA it is supposed to sign.

A subkey consists of two files, a private key pair in a .pem file and the
signed public key in a .bin file. Subkeys reuse the signed header (SHDR)
format used for signed TAs, followed by a payload holding a public key and
UUID among other fields. A subkey is formatted with all fields in
little endian as:

+-----------+-------------------+---------------------+-----------------------+
| Size in   | Field Name        | Protected by field  |                       |
| bytes     |                   |                     |                       |
+-----------+-------------------+---------------------+-----------------------+
| .. centered:: Signed header (struct shdr)           |                       |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   magic           |  hash               | 0x4f545348            |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   img_type        |  hash               | 3, SHDR_SUBKEY        |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   img_size        |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   algo            |  hash               | Signature algorithm,  |
|           |                   |                     | GP TEE_ALG_*          |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   hash_size       |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   sig_size        |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| hash_size |   hash            |  sig                |                       |
+-----------+-------------------+---------------------+-----------------------+
| sig_size  |   sig             |  previous pub key   | Root key or a subkey  |
|           |                   |                     | higher up in the      |
|           |                   |                     | chain                 |
+-----------+-------------------+---------------------+-----------------------+
| .. centered:: Subkey header (struct shdr_subkey)    |                       |
+-----------+-------------------+---------------------+-----------------------+
| 16        |   UUID            |  hash               | UUIDv5 namespace for  |
|           |                   |                     | next subkey or TA     |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   name_size       |  hash               | Size of the UUIDv5    |
|           |                   |                     | name for the          |
|           |                   |                     | next subkey or TA,    |
|           |                   |                     | 0 for identiy subkeys |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   subkey_version  |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   max_depth       |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   algo            |  hash               | Signature algorithm   |
|           |                   |                     | for the next          |
|           |                   |                     | subkey or TA          |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   attr_count      |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
|           | .. centered:: Subkey attrs * attr_count | Attributes of the     |
|           |                                         | public key in this    |
|           |                                         | subkey                |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   id              |  hash               | GP TEE_ATTR_*         |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   offs            |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| 4         |   size            |  hash               |                       |
+-----------+-------------------+---------------------+-----------------------+
| opaque data padding up this   |  hash               | The subkey attributes |
| binary to img_size            |                     | above point into      |
|                               |                     | this area using       |
|                               |                     | offset and size       |
+-------------------------------+---------------------+-----------------------+

All subkeys included in the subkey hierarchy are added in front when a TA
is signed using a subkey. For example, if a TA is signed using the
second-level subkey above it would look like this:

+------------------+----------------------+-----------------------------------+
| Size in bytes    | Binary               |                                   |
+------------------+----------------------+-----------------------------------+
| first.img_size   | First-level subkey   | Signed by Root key                |
+------------------+----------------------+-----------------------------------+
| first.name_size  | UUIDv5 name string   | Not signed, used to prove that    |
|                  | for the next subkey  | the next UUID is in the namespace |
|                  |                      | of the First-level subkey.        |
|                  |                      | This size is from "name_size"     |
|                  |                      | of the previous subkey            |
|                  |                      | (First-level).                    |
+------------------+----------------------+-----------------------------------+
| second.img_size  | Second-level subkey  | Signed by First-level subkey      |
+------------------+----------------------+-----------------------------------+
| second.name_size | UUIDv5 name string   | Not signed used to prove that     |
|                  | for the TA           | the next UUID is in the namespace |
|                  |                      | of the Second-level subkey.       |
|                  |                      | This size is from "name_size"     |
|                  |                      | of the previous subkey            |
|                  |                      | (Second-level).                   |
+------------------+----------------------+-----------------------------------+
| second.img_size  | TA                   | Signed by Second-level subkey     |
+------------------+----------------------+-----------------------------------+

The signed TA binary is self-contained with all the public keys
needed for verification included, except the public root key which is
embedded in the TEE core binary.

The UUIDv5 name string is a separate field between subkeys and the next
subkey or TA to allow a subkey to be used to sign more than one other
subkey or TA.

A signed TA or subkey can be inspected using the sign_encrypt.py script,
for example::

    $ scripts/sign_encrypt.py display --in 5c206987-16a3-59cc-ab0f-64b9cfc9e758.ta
    Subkey
     struct shdr
      magic:      0x4f545348
      img_type:   3 (SHDR_SUBKEY)
      img_size:   320 bytes
      algo:       0x70414930 (TEE_ALG_RSASSA_PKCS1_PSS_MGF1_SHA256)
      hash_size:  32 bytes
      sig_size:   256 bytes
      hash:       f573f329fe77be686ce71647909c4ea35b5e1cd7de86369bd7d9fca31f6a4d65
     struct shdr_subkey
      uuid:       f04fa996-148a-453c-b037-1dcfbad120a6
      name_size:  64
      subkey_version: 1
      max_depth:  4
      algo:       0x70414930 (TEE_ALG_RSASSA_PKCS1_PSS_MGF1_SHA256)
      attr_count: 2
      next name:  "mid_level_subkey"
    Next header at offset: 692 (0x2b4)
    Subkey
     struct shdr
      magic:      0x4f545348
      img_type:   3 (SHDR_SUBKEY)
      img_size:   320 bytes
      algo:       0x70414930 (TEE_ALG_RSASSA_PKCS1_PSS_MGF1_SHA256)
      hash_size:  32 bytes
      sig_size:   256 bytes
      hash:       233a6dcf1a2cf69e50cde8e20c4129157da707c76fa86ce12ee31037edef02d7
     struct shdr_subkey
      uuid:       1a5948c5-1aa0-518c-86f4-be6f6a057b16
      name_size:  64
      subkey_version: 1
      max_depth:  3
      algo:       0x70414930 (TEE_ALG_RSASSA_PKCS1_PSS_MGF1_SHA256)
      attr_count: 2
      next name:  "subkey1_ta"
    Next header at offset: 1384 (0x568)
    Bootstrap TA
     struct shdr
      magic:      0x4f545348
      img_type:   1 (SHDR_BOOTSTRAP_TA)
      img_size:   84576 bytes
      algo:       0x70414930 (TEE_ALG_RSASSA_PKCS1_PSS_MGF1_SHA256)
      hash_size:  32 bytes
      sig_size:   256 bytes
      hash:       ea31ac7dc2cc06a9dc2853cd791dd00f784b5edc062ecfa274deeb66589b4ca5
     struct shdr_bootstrap_ta
      uuid:       5c206987-16a3-59cc-ab0f-64b9cfc9e758
      ta_version: 0
     TA offset:  1712 (0x6b0) bytes
     TA size:    84576 (0x14a60) bytes


.. rubric:: Footnotes

.. [#f1] UUIDv5 and namespaces are described in
	`RFC4122 <https://datatracker.ietf.org/doc/html/rfc4122>`_.
	Note that OP-TEE uses a truncated SHA-512 instead of the
        weak SHA-1 hash when when deriving a new UUID from a
        namespace and name.
