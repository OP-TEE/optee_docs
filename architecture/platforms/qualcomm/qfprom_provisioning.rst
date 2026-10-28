.. _qualcomm_qfprom_provisioning:

########################
QFPROM fuse provisioning
########################

The :ref:`qualcomm_hoya` chipsets use QFPROM for one-time-programmable fuse
storage. OP-TEE provides fuse reads and a boot provisioning service for
programming device identity, key material and security policy.

Architecture and platform support
*********************************

The boot loader supplies the provisioning data from a ``sec.elf`` image in
non-secure memory before OP-TEE starts. OP-TEE validates the request, manages
the resources needed for programming, and applies the selected fuse settings.
Restricted fuse reads for PAS authentication are a separate service and do
not require boot provisioning.

``CFG_QCOM_QFPROM_FUSEPROV`` controls boot provisioning and defaults to enabled
on secure builds. Both Kodiak and Lemans use Command DB to identify shared
power resources and RPMh to request the programming supplies. Kodiak also
requires an additional MX-rail vote, coordinated with the Always-On Processor
(AOP). Ordinary fuse reads do not require the programming supply sequence.

Image generation and OEM signing
********************************

The `Qualcomm security tools wrapper
<https://github.com/qualcomm-linux/qcom-sec-tools-wrapper>`_ provides scripts
for generating fuse provisioning images (``sec.elf``) and OEM signing of
firmware images using Qualcomm SecTools. Its README covers tool setup,
target security profiles, signing inputs, and fuse-selection examples.

The preceding boot loader is responsible for authenticating the provisioning
image according to the device's security policy and making its SecDat payload
available to OP-TEE. OP-TEE checks the payload's structure and SHA-256 checksum
before processing the fuse request. This integrity check does not establish
the image's signing identity.

Provisioning behavior
*********************

Provisioning is skipped in download mode. Outside download mode, OP-TEE uses
a private copy of the supplied data so that changes in non-secure memory
cannot alter the request during processing. Unsupported or malformed requests
are rejected before programming resources are enabled.

Programming is serialized, with the provisioning lock checked before writes.
Key material and configuration are programmed before the final error-correction
and read/write-permission settings. Permission locks and other security settings
must be included in the provisioning plan; requesting key programming alone
does not lock access to the key region.

For the secondary hardware key (SHK), OP-TEE generates random key material
on the device using the hardware random number generator. It uses this
material instead of the key values carried in the image and applies the
required forward error correction (FEC). Existing SHK data prevents key
regeneration. A fully locked SHK region is skipped; inconsistent read/write
permissions cause provisioning to fail. OEM-spare provisioning
also supports device-generated random data and checks for existing data
before its random-data programming pass.

Readable regions are checked after programming. Private regions whose
contents are hidden by hardware cannot be verified by comparing their data.
FEC errors encountered while reading are reported as failures.

After a programming attempt, OP-TEE disables further writes, releases its
programming supply requests and restores the clock configuration. Cleanup
also runs on failure. Temporary key material and the consumed provisioning
input are cleared after use.

Boot outcomes and recovery
**************************

An already locked device, absent provisioning data, or a valid image without
fuse operations allows boot to continue without programming. A provisioning
failure stops boot with a panic after cleanup. A failure may leave some fuses
programmed: writes are irreversible and the service cannot roll them back.
Existing-data checks prevent automatic SHK regeneration and can prevent a
partially completed request from being retried.

After successful fuse writes, OP-TEE reports that a manual reset from U-Boot
is required. It does not reset the device automatically. The reset is part
of completing the provisioning workflow.
