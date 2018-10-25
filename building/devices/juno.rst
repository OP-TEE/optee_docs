.. _juno:

####
Juno
####
The instructions here will tell how to run OP-TEE on the Juno board. The
instructions has been tested and verified on the ``Juno r0`` revision (see `Juno
revisions`_ for more details).

Regular build
*************
First step is to start out by following the instructions in the
:ref:`get_and_build_the_solution` as described in :ref:`build`.

Deploy files on the device
**************************
Enter the firmware console on the Juno board and press **enter** to stop the
auto boot.

.. code-block:: bash

    ARM V2M_Juno Firmware v1.3.9
    Build Date: Nov 11 2015

    Time :  12:50:45
    Date :  29:03:2016

    Press Enter to stop auto boot...

Enable ``FTP`` at the firmware prompt.

.. code-block:: bash

    Cmd> ftp_on
    Enabling ftp server...
     MAC address: xxxxxxxxxxxx

     IP address: 192.168.1.158

     Local host name = V2M-JUNO-A2

Flash the binary by running

.. note::
    Use the **IP address** from output from previous command.

.. code-block:: bash

    $ make JUNO_IP=192.168.1.158 flash

Once all binaries have been transferred, reboot the board:

.. code-block:: bash

    Cmd> reboot

Update the flash layout
***********************
The flash layout for Juno may need to be updated for the flashing above to work.
If flashing fails or if TF-A refuses to boot due to wrong version of the ``SCP``
binary, then the flash(-layout) needs to be updated. To update the flash please
follow the instructions at Arm's `old release notes`_ page selecting one of the
zips under "Development boards / Juno / Prebuilt configurations" and flash it as
described at `Run the Arm Platforms deliverables on Juno`_.

GlobalPlatform testsuite support
********************************
.. note::

    Depending on the Juno pre-built configuration, the built ``ramdisk.img`` size
    with GlobalPlatform testsuite may exceed its pre-defined Juno flash memory
    reserved location (``image.txt`` file). In that case, you will need to extend
    the Juno flash block size reserved location for the ``ramdisk.img`` in the
    ``image.txt`` file accordingly and follow the instructions under "5.7.1 Update
    flash and its layout".

Example
=======
Example with ``juno-latest-busybox-uboot.zip``. The current ``ramdisk.img`` size
with GlobalPlatform testsuite is 8.6 MBytes and that is too big to fit in the
default configuration, therefore we need to make adjustments to the flash
layout. You will do that by making changes to
``/JUNO/SITE1/HBI0262B/images.txt``. I.e., from:

.. code-block:: none
    :emphasize-lines: 2
    :linenos:

    NOR4UPDATE: AUTO                 ;Image Update:NONE/AUTO/FORCE
    NOR4ADDRESS: 0x01800000          ;Image Flash Address
    NOR4FILE: \SOFTWARE\ramdisk.img  ;Image File Name
    NOR4NAME: ramdisk.img
    NOR4LOAD: 00000000               ;Image Load Address
    NOR4ENTRY: 00000000              ;Image Entry Point

to extending the *Image Flash Address* to 16MB

.. code-block:: none
    :emphasize-lines: 2
    :linenos:

    NOR4UPDATE: AUTO                 ;Image Update:NONE/AUTO/FORCE
    NOR4ADDRESS: 0x01000000          ;Image Flash Address
    NOR4FILE: \SOFTWARE\ramdisk.img  ;Image File Name
    NOR4NAME: ramdisk.img
    NOR4LOAD: 00000000               ;Image Load Address
    NOR4ENTRY: 00000000              ;Image Entry Point

GCC > 5.x support
*****************

.. note::
    In case you are using the **latest version** of the OP-TEE Arm Juno build
    (i.e., ``juno.xml`` manifest), then the ``ramdisk.img`` built with a GCC
    version newer than 5.x will be bigger than built with older GCC versions.
    This means that you will need to update the sections in ``image.txt`` that
    tells where various images will start (see the ``image.txt`` file).

To solve this problem you will need to extend the Juno flash block size reserved
location for the ``ramdisk.img`` and decrease the size for other images in the
``image.txt`` file accordingly in the same manner as described in the previous
section above.

For example with ``juno-latest-busybox-uboot.zip``. The current ``ramdisk.img``
size with GCC 5.x compiler is 29.15MB and therefore we will need to extend that
size for that to 32MB. You do that by changing the highlighted ones (i.e.,
*Image Flash Address*) in file ``/JUNO/SITE1/HBI0262B/images.txt``.

.. code-block:: none
    :emphasize-lines: 2, 9, 16, 23
    :linenos:

    NOR2UPDATE: AUTO                 ;Image Update:NONE/AUTO/FORCE
    NOR2ADDRESS: 0x00100000          ;Image Flash Address
    NOR2FILE: \SOFTWARE\Image        ;Image File Name
    NOR2NAME: norkern                ;Rename kernel to norkern
    NOR2LOAD: 00000000               ;Image Load Address
    NOR2ENTRY: 00000000              ;Image Entry Point

    NOR3UPDATE: AUTO                 ;Image Update:NONE/AUTO/FORCE
    NOR3ADDRESS: 0x02C00000          ;Image Flash Address
    NOR3FILE: \SOFTWARE\juno.dtb     ;Image File Name
    NOR3NAME: board.dtb              ;Specify target filename to preserve file extension
    NOR3LOAD: 00000000               ;Image Load Address
    NOR3ENTRY: 00000000              ;Image Entry Point

    NOR4UPDATE: AUTO                 ;Image Update:NONE/AUTO/FORCE
    NOR4ADDRESS: 0x00D00000          ;Image Flash Address
    NOR4FILE: \SOFTWARE\ramdisk.img  ;Image File Name
    NOR4NAME: ramdisk.img
    NOR4LOAD: 00000000               ;Image Load Address
    NOR4ENTRY: 00000000              ;Image Entry Point

    NOR5UPDATE: AUTO                 ;Image Update:NONE/AUTO/FORCE
    NOR5ADDRESS: 0x02D00000          ;Image Flash Address
    NOR5FILE: \SOFTWARE\hdlcdclk.dat ;Image File Name
    NOR5LOAD: 00000000               ;Image Load Address
    NOR5ENTRY: 00000000              ;Image Entry Point

.. _Juno revisions: https://community.arm.com/dev-platforms/w/docs/253/juno-board-revisions
.. _old release notes: https://community.arm.com/dev-platforms/w/docs/226/old-release-notes
.. _Run the Arm Platforms deliverables on Juno: https://community.arm.com/dev-platforms/w/docs/391/run-the-arm-platforms-deliverables-on-juno
