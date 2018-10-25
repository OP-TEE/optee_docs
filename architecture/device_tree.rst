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
      memory.

    - Boot loader stages may load a device tree structure in secure memory for
      the benefit of the secure world only. Such device tree blob shall be
      located in secure memory.

    - OP-TEE core can also embedded a device tree structure to describe the
      platform.

    - Non-secure world can embed a device tree structure and/or rely on a device
      tree structure loaded by the secure world, being an early boot stage
      and/or OP-TEE core.

Obviously the non-secure world will not be able to access a device tree image
located on a secure memory which non-secure world as no access to.

Early boot device tree argument
*******************************
The bootloader provides arguments to the OP-TEE core when it boots it. Among
those, the physical memory base address of a device tree image accessible to
OP-TEE core.

When OP-TEE core is built with ``CFG_DT=y`` this device tree is accessed by
OP-TEE core to get some information: console configuration, main memory size.

OP-TEE will also try to add the description of the OP-TEE resources for the
non-secure world to properly communicate with OP-TEE. This assumes the image is
located in non-secure memory.

Modifications made by OP-TEE core on the non-secure device tree image provided
by early boot and passed to non-secure world are the following:

    - Add an OP-TEE node if none found with the related invocation parameters.

    - Add a reserved memory node for the few memory areas that shall be reserved
      to the secure world and non accessed by the non-secure world.

    - Add a PSCI description node if none found.

Early boot DTB located in non-secure memory can be accessed by OP-TEE core only
during its initialization, before non-secure world boots.

Embedded Secure Device Tree
***************************
When OP-TEE core is built with configuration directive ``CFG_EMBED_DTB=y``
directive ``CFG_EMBED_DTB_SOURCE_FILE`` shall provide the relative path of the
DTS file inside directory ``core/arch/$(ARCH)/dts`` from which a DTB is
generated and embedded in a read-only section of OP-TEE core.

In this case the device tree address passed to the OP-TEE entry point by the
bootloader is ignored, only the embedded device tree is accessible.
