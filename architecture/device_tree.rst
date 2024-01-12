.. _device_tree:

###########
Device Tree
###########
OP-TEE core can use the device tree format to inject platform configuration
information during platform initialization and possibly some run time contexts.

Device Tree technology allows to describe platforms from ASCII source files
so-called DTS files. These can be used to generate a platform description binary
image, so-called DTB, embedded in the platform boot media for applying expected
configuration settings during the platform initializations.

This scheme relaxes design constraints on the OP-TEE core implementation as most
of the platform specific hardware can be tuned without modifying C source files
or adding configuration directives in the build environments.

Secure and Non-Secure Device Trees
**********************************
There can be several device trees embedded in the target system and some can be
shared across the boot stages.

    - Boot loader stages may load a device tree structure in memory for all boot
      stage to get platform configuration from. If such device tree data are to
      be accessed by the non-secure world, they shall be located in non-secure
      memory. Secure world may use its content during OP-TEE core
      initialization.

    - Boot loader stages may load a device tree structure in secure memory for
      the benefit of the secure world only. Such device tree blob shall be
      located in secure memory. Secure world could use its content but this
      is currently not implemented in the latest OP-TEE release.

    - OP-TEE core can also embedded a device tree structure to describe the
      platform.

    - Non-secure world can embed its own device tree structure(s) and/or
      rely on a device tree structure loaded by the secure world during
      its initialization which happen before non-secure world is booted.

Obviously the non-secure world will not be able to access a device tree image
located in a secure memory which non-secure world has no access to.

When OP-TEE core is built with ``CFG_DT=y``, non-secure and secure device trees
can be accessed by OP-TEE core to get some platform configuration information.

.. _generic_boot_and_dtbs:

Generic boot and DTBs
*********************
Generic boot sequence gets discovers main memory address ranges from
preferrably embedded DTB (section :ref:`embedded_dtb`), defaulting to
early boot external DTB (section :ref:`external_dtb`).

Generic boot uses early boot external DTB (section :ref:`external_dtb`)
to share platform configuration information with the non-secure world.

Plaform and drivers can call OP-TEE DT API (``core/include/kernel/dt.h``)
to access embedded and/or external DTBs.

.. _external_dtb:

Early boot external device tree
*******************************
The bootloader provides arguments to OP-TEE core when it boots it. Among
those, the physical memory base address of a non-secure device tree image
accessible to OP-TEE core, or a null address value in absence of such DTB.

Platform configuration may statically define such DTB location using the
build configuration directive ``CFG_DT_ADDR``.

When an external DTB is referred, OP-TEE core gets the console configuration
if the platform has registered a compatible driver by adding attribute
``__dt_driver`` to a defined ``const struct dt_driver`` instance.

When an external DTB is referred, OP-TEE core adds into this DTB the
description of some OP-TEE resources. These information can be used
by the non-secure world to properly communicate with OP-TEE. This scheme
assumes the image is located in non-secure memory.

Modifications made by OP-TEE core on the non-secure device tree image provided
by early boot and passed to non-secure world are the following:

    - Add an OP-TEE node if none found with the related invocation parameters.

    - Add a reserved memory node for the few memory areas that shall be reserved
      to the secure world and non accessed by the non-secure world.

    - Add a PSCI description node if none found.

Early boot DTB can be accessed by OP-TEE core only during its initialization,
before non-secure world boots as it is expected the DTB memory location has
likely been replaced with runtime contexts content.

Assuming there is no embedded DTB (section :ref:`embedded_dtb`) OP-TEE core
discovers the main memory address ranges from the non-secure DTB.

.. _external_dtb_overlay:

Early boot device tree overlay
******************************
There are two possibilities for OP-TEE core to provide a device tree
overlay to the non-secure world.

    - Append OP-TEE nodes to an existing DTB overlay located in early boot DTB.
      (``CFG_DT_ADDR`` or boot argument register ``R2``/``X2``).

    - Generate a new DTB overlay image at location defined by ``CFG_DT_ADDR``.

In the later case, memory referred by configuration directive ``CFG_DT_ADDR``
shall not contain a valid DTB image when OP-TEE core is booted. A subsequent
non-secure boot stage should merge the OP-TEE DTB overlay image into
another DTB.

A typical bootflow for this would be Trusted Firmware-A -> OP-TEE -> U-Boot
with U-Boot in charge of merging OP-TEE DTB overlay located at ``CFG_DT_ADDR``
into a DTB U-Boot has loaded from elsewhere.

This functionality is enabled when ``CFG_EXTERNAL_DTB_OVERLAY=y``.

.. _embedded_dtb:

Embedded Secure Device Tree
***************************
When OP-TEE core is built with configuration directive ``CFG_EMBED_DTB=y``,
directive ``CFG_EMBED_DTB_SOURCE_FILE`` shall provide the relative path of the
DTS file inside directory ``core/arch/$(ARCH)/dts`` from which a DTB is
generated and embedded in a read-only section of OP-TEE core.

Refer to ``core/include/kernel/dt.h`` for API to access embedded DTB.

Section :ref:`generic_boot_and_dtbs` documents the generic boot sequence
against embedded DTB.

.. _optee_specific_bindings:

OP-TEE Specific Bindings
***************************
:ref:`google_widevine_bindings`
