.. _fvp:

###
FVP
###
The instructions here will tell how to build and run OP-TEE using Foundation
Models.


.. _fvp_build_instructions:

Build instructions
******************
Start out by following the ":ref:`get_and_build_the_solution`" as described in
:ref:`build`. However, stop before doing ":ref:`build_make`".

Next you should obtain the `Armv8-A Foundation Platform (For Linux Hosts
Only)`_. To download FVPs you’ll need to log in to Arm Self Service. That binary
should be untar'ed to the root of the repo forest, i.e., like this:
``<fpv-project>/Foundation_Platformpkg``. In the end after cloning all source
code, getting the toolchains and "installing" Foundation_Platformpkg you should
have a folder structure that looks like this:

.. code-block:: none
    :emphasize-lines: 9

    $ ls -al
    drwxrwxr-x 15 jbech jbech 4096 Feb  5 09:10 .
    drwxr-xr-x 22 jbech jbech 4096 Jan 15 12:45 ..
    drwxrwxr-x 18 jbech jbech 4096 Feb  5 09:10 arm-trusted-firmware
    drwxrwxr-x  9 jbech jbech 4096 Feb  5 09:10 build
    drwxrwxr-x 15 jbech jbech 4096 Feb  5 09:10 buildroot
    drwxrwxr-x 51 jbech jbech 4096 Feb  5 09:10 edk2
    drwxrwxr-x  5 jbech jbech 4096 Feb  5 09:10 edk2-platforms
    drwxrwxr-x  6 jbech jbech 4096 Mar 15  2018 Foundation_Platformpkg
    drwxrwxr-x 15 jbech jbech 4096 Feb  5 09:10 grub
    drwxrwxr-x 26 jbech jbech 4096 Feb  5 09:10 linux
    drwxrwxr-x  6 jbech jbech 4096 Feb  5 09:10 optee_client
    drwxrwxr-x 10 jbech jbech 4096 Feb  5 09:10 optee_examples
    drwxrwxr-x 11 jbech jbech 4096 Feb  5 09:10 optee_os
    drwxrwxr-x  8 jbech jbech 4096 Feb  5 09:10 optee_test
    drwxrwxr-x  7 jbech jbech 4096 Feb  5 09:10 .repo
    lrwxrwxrwx  1 jbech jbech   23 Feb  5 09:09 toolchains

When this pre-condition met you can simply continue with

.. code-block:: bash

    $ make run

and then FVP should build the rootfs and then start the simulation and when you
have a terminal you can log in and run xtest (as described at
:ref:`build_run_xtest`).

.. _Armv8-A Foundation Platform (For Linux Hosts Only): https://developer.arm.com/products/system-design/fixed-virtual-platforms
