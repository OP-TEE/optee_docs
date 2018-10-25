.. todo::

    Joakim: Question is whether this should be moved under the "optee_os"
    specific page instead?

.. _file_structure:

File structure
##############
This page describes what different folders in :ref:`optee_os` contains.

Top level directories
*********************
.. list-table:: Top level directories
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /core
      - Files that are only used building TEE Core

    * - /lib
      - Files that are used both when building TEE Core and TAs

    * - /ta
      - Files that are only used when building TAs

    * - /mk
      - Makefiles supporting the build system

    * - /tmp-stuff
      - Temporary stuff that will be removed before the final commit is made

    * - /scripts
      - Helper scripts for miscellaneous tasks

    * - /out
      - Created when building unless a different out directory is specified with
        ``O=...`` on the command line

/core
*****
.. list-table:: Structure of /core
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /arch
      - Architecture and platform specific files

    * - /include
      - Header files of resources exported by the core

    * - /lib
      - Generic libraries that are likely to be replaced in a final product

    * - /mm
      - Generic memory management, currently empty

    * - /tee
      - Generic TEE files

/core/arch
**********
.. list-table:: Structure of /core/arch
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /arm
      - ARMv7 and Aarch32 specific architecture and platform specific files

/core/arch/arm
**************
.. list-table:: Structure of /core/arch/arm
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /dts
      - Device tree source files

    * - /include
      - Include files used in rest of TEE core but not in any supporting
        libraries

    * - /kern
      - Low level and core parts of TEE Core

    * - /mm
      - Memory management

    * - /tee
      - TEE files

    * - /sm
      - Secure Monitor

    * - /plat-foo
      - Specific files for the ``foo`` platform

/core/arch/arm/include
**********************
.. list-table:: Structure of /core/arch/arm/include
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /kern
      - Include files exposing API for /core/arch/arm/kern files

    * - /kta
      - Include files exposing the KTA API that is mainly used by kernel TAs

    * - /mm
      - Include files exposing API for /core/arch/arm/mm files

    * - /rom
      - Old ROM files that should be removed before going public

    * - /sm
      - Include files exposing API for Secure Monitor

/core/include
*************************
.. list-table:: Structure of /core/include
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /drivers
      - Include files exposing API for /core/drivers files

    * - /dt-bindings
      - Include files for the device tree bindings

/core/lib/lib{crypto,sla}
*************************
.. list-table:: Structure of /core/lib/lib{crypto,sla}
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /
      - Source files for the library

    * - /include  
      - Include files exposing the API of the library

/lib/libutils
*************
.. list-table:: Structure of /lib/libutils
    :header-rows: 1
    :widths: 1 5

    * - Directory
      - Description

    * - /
      - Source file for the library

    * - /arch	  
      - Architecture specific source files

    * - /arch/arm 
      - ARMv7 and Aarch32 specific source files

    * - /arch/arm/include 
      - ARMv7 and Aarch32 specific include files

    * - /include  
      - Include files exposing the API of the library
