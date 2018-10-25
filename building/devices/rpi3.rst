.. _rpi3:

##############
Raspberry Pi 3
##############
`Sequitur Labs`_ did the initial OP-TEE port which at the time also came with
modifications in U-Boot, Trusted Firmware A and Linux kernel. Since that initial
port more and more patches have found mainline trees and today the OP-TEE setup
for Raspberry Pi 3 uses only upstream tree's with the exception of Linux kernel.


Disclaimer
**********
.. warning::

    This port of Trusted Firmware A and OP-TEE to Raspberry Pi 3 **IS NOT
    SECURE!** Although the Raspberry Pi3 processor provides ARM TrustZone
    exception states, the mechanisms and hardware required to implement secure
    boot, memory, peripherals or other secure functions are not available. Use
    of OP-TEE or TrustZone capabilities within this package **does not result**
    in a secure implementation. This package is provided solely for
    **educational purposes** and **prototyping**.


.. _rpi3_software:

What is expected to work?
*************************
First, note that all OP-TEE developer builds (ref, :ref:`build`) have rather
simple overall goals:

    - Successfully build OP-TEE for certain devices.
    - Run xtest and optee_example binaries successfully with no regressions
      using UART(s).

I.e., it is important to understand that our "`OP-TEE developer builds`" shall
not be compared with full Linux distributions which supports "everything". As a
couple of examples, we don't enable any particular drivers in Linux kernel, we
don't include all sorts of daemons, we do not include an X-environment etc. At
the same time this doesn't mean that you cannot use OP-TEE in real environments.
It is usually perfectly fine to run on all sorts of devices, environments etc.
It's just that for the OP-TEE developer builds we have intentionally stripped
down the environment to make it rather fast to get all the source code, build it
all and run xtest.

We are highlighting this here, since over the years we have had many questions
at GitHub about things that people usually find working on their Raspberry Pi
devices when they are using Raspbian (which this is not). The table below
describes what is `officially` supported in the Raspberry Pi 3 OP-TEE developer
builds and right after that follows sections for each of giving a bit more
context to it.

    +-----------------+------------+
    | Name            | Supported? |
    +=================+============+
    | Buildroot       | Yes        |
    +-----------------+------------+
    | HDMI            | No         |
    +-----------------+------------+
    | NFS             | Yes        |
    +-----------------+------------+
    | Random packages | Maybe      |
    +-----------------+------------+
    | Raspbian        | No         |
    +-----------------+------------+
    | Secure boot     | Maybe      |
    +-----------------+------------+
    | TFTP            | Yes        |
    +-----------------+------------+
    | UART            | Yes        |
    +-----------------+------------+
    | Wi-Fi           | No         |
    +-----------------+------------+


.. _rpi3_support_buildroot:

Buildroot
=========
We are using Buildroot as the tool to create a stripped down filesystem for
Linux where we also put OP-TEE binaries like Trusted Applications, client
libraries and TEE supplicant. If a user wants to add/enable additional packages,
then that is also possible by adding new lines in ``common.mk`` in :ref:`build`
(search for ``BR2_PACKAGE_`` in the git to see how it's done).


.. _rpi3_support_hdmi:

HDMI
====
X isn't enabled and we have not built nor enabled any drivers for graphics.


.. _rpi3_support_nfs:

NFS
===
Works to boot up a Linux root filesystem, more on that further down.


.. _rpi3_support_random_package:

Random packages
===============
See the :ref:`rpi3_support_buildroot` section above. You can enable packages
supported by Buildroot, but as mentioned initially in this section, lack of
drivers and other daemons etc might make it impossible to run.


.. _rpi3_support_raspbian:

Raspbian
========
We are not using it. However, people (from `Sequitur Labs`_) have successfully
been able to add OP-TEE to Raspbian builds. But since we're not using it and
haven't tried, we simply don't support it.


.. _rpi3_support_secure_boot:

Secure boot
===========
First pay attention to the initial warning on this page. I.e., no matter what
you are doing with Raspberry Pi and TrustZone / OP-TEE you **cannot** make it
secure. But that doesn't mean that you cannot "enable" secure features as such
for prototyping and to learn how to build and use those. That kind of knowledge
can later on be transferred and used on other devices which have all the
necessary secure capabilities needed to make a secure system. We haven't tested
to enable secure boot on Raspberry Pi 3. But we believe that a good starting
point would be Trusted Firmware A's documentation about the "`Authentication
Framework`_" and `RPi3 in TF-A`_.


.. _rpi3_support_tftp:

TFTP
====
When you reach U-Boot (see :ref:`rpi3_boot_sequence`), then you can start using
TFTP to load boot firmware etc. Note that if you overwrite ``armstub8.bin`` for
example and that happens to be faulty, then you will need to re-mount the BOOT
partition on the SD-card and put a new working version of it. Also note that
changing early boot binaries (TF-A, OP-TEE core etc) will require you to reboot
the device see the changes.


.. _rpi3_support_uart:

UART
====
Fully supported, for more details look at the UART section further down.


.. _rpi3_support_wifi:

Wi-Fi
=====
Even though Raspberry Pi 3 has a Wi-Fi chip, we do not support it in our
stripped down builds.


.. _rpi_hardware:

What versions of Raspberry Pi will work?
****************************************
Below is a table of supported hardware in our OP-TEE developer builds. We have
only used the Raspberry Pi 3 Model B, i.e., the first RPi 3 device that was
released. But we know that people have successfully been able to use it with
both RPi 2's as well as the newer RPi 3 B+. But as long as we in the `core
team`_ doesn't have those at hands we cannot guarantee anything, therefore we
simply say "No" below.

    +-------------------------------+------------+
    | Hardware                      | Supported? |
    +===============================+============+
    | Raspberry Pi 1 Model A        | No         |
    +-------------------------------+------------+
    | Raspberry Pi 1 Model B        | No         |
    +-------------------------------+------------+
    | Raspberry Pi 1+ Model A       | No         |
    +-------------------------------+------------+
    | Raspberry Pi 1+ Model B       | No         |
    +-------------------------------+------------+
    | Raspberry Pi 2 Model B        | No         |
    +-------------------------------+------------+
    | Raspberry Pi 2 Model B v1.2   | No         |
    +-------------------------------+------------+
    | Raspberry Pi 3+ Model A       | No         |
    +-------------------------------+------------+
    | Raspberry Pi 3 Model B        | Yes        |
    +-------------------------------+------------+
    | Raspberry Pi 3+ Model B       | No         |
    +-------------------------------+------------+
    | Zero - all versions           | No         |
    +-------------------------------+------------+
    | Compute module - all versions | No         |
    +-------------------------------+------------+


.. _rpi3_boot_sequence:

Boot sequence
*************
    - The **GPU** starts executing the first stage bootloader, which is stored
      in ROM on the SoC. The first stage bootloader reads the SD-card, and loads
      the second stage bootloader (``bootcode.bin``) into the L2 cache, and runs
      it.
    - ``bootcode.bin`` enables SDRAM, and reads the third stage bootloader
      ``loader.bin`` from the SD-card into RAM, and runs it.
    - ``loader.bin`` reads the GPU firmware (``start.elf``).
    - ``start.elf`` reads ``config.txt``, pre-loads ``armstub8.bin`` (which
      contains: BL1/TF-A + BL2/TF-A + BL31/TF-A + BL32/OP-TEE + BL33/U-boot) to
      ``0x0`` and jumps to the first instruction.
    - A traditional boot sequence of TF-A -> OP-TEE -> U-boot is performed,
      i.e.,  BL1 loads BL2, then BL2 loads and run BL31(SM), BL32(OP-TEE),
      BL33(U-boot) (one after another)
    - U-Boot runs ``fatload/booti`` sequence  to load from eMMC to RAM both
      ``zImage`` and then ``DTB`` and boot.


.. _rpi3_build_instructions:

Build instructions
******************
1. Start by following the :ref:`get_and_build_the_solution` as described in
   :ref:`build`, but stop at the ":ref:`build_flash`" step (i.e., **don't** run
   the make flash command!).

2. Next step is to partition and format the memory card and to put the files
   onto the same. That is something we don't want to automate, since if anything
   goes wrong, in worst case it might wipe one of your regular hard disks.
   Instead what we have done, is that we have created another makefile target
   that will tell you exactly what to do. Run that command and follow the
   instructions there.

   .. code-block:: bash

        $ make img-help

   .. note::

       The mention of ``/dev/sdx1`` and ``/dev/sdx2`` when running the command
       above are just examples. You need to figure out and replace that with the
       correct name(s) for your computer and SD-card (typically run ``dmesg``
       and look for the device name matching your SD-card).

3. Put the SD-card back into the Raspberry Pi 3.

4. Plug in the UART cable and attach to the UART

    .. code-block:: bash

        $ picocom -b 115200 /dev/ttyUSB0

    .. note::

        Install picocom if not already installed ``$ sudo apt-get install picocom``.

5. Power up the Raspberry Pi 3 and the system shall start booting which you will
   see on the UART (not :ref:`rpi3_support_hdmi`).

6. When you have a shell, then it's simply just to follow the
   ":ref:`build_run_xtest`" instructions.


.. _rpi3_nfs:

NFS boot
********
Booting via NFS is quite useful for several reasons, but the obvious reason when
working with Raspberry Pi is that you don't have to move the SD-card back and
forth between the host machine and the Raspberry Pi 3 itself when working with
**Normal World** files, like Linux kernel and user space programs. Here we will
describe how to setup NFS server, so the rootfs can be mounted via NFS.

.. warning::

    This guide doesn't focus on any desktop security, so eventually you would
    need to harden your setup.

In the description below we will use the following terminology, IP addresses and
paths. The reader of this guide is supposed to update this to match his own
environment.

.. code-block:: none

    192.168.1.100   <--- This is your desktop computer (NFS server)
    192.168.1.200   <--- This is the Raspberry Pi
    /srv/nfs/rpi    <--- Location for the NFS share


Configure NFS
=============
Start by installing the NFS server

.. code-block:: bash

    $ sudo apt-get install nfs-kernel-server

Then edit the exports file,

.. code-block:: bash

    $ sudo vim /etc/exports

In this file you shall tell where your files/folder are and the IP's allowed to
access the files. The way it's written below will make it available to every
machine on the same subnet (again, be careful about security here). Let's add
this line to the file (it's the only line necessary in the file, but if you have
several different filesystems available, then you should of course add them
too, one line for each share).

.. code-block:: none

    /srv/nfs/rpi 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)

Next create the folder where you are going to put the root filesystem

.. code-block:: none

    $ sudo mkdir /srv/nfs/rpi

After this, restart the NFS kernel server

.. code-block:: none

    $ service nfs-kernel-server restart

.. hint::

    To see that your shares are correctly setup and that the NFS server is
    running, you can run: ``$ showmount --all localhost`` and you should get a
    list of ``IP:<path>'s`` based on what you have added in your exports file.
    If you get nothing there, then your NFS server hasn't been setup correctly.

Prepare files to be shared
==========================
We are now going to put the root filesystem on the location we prepared in the
previous section.

.. note::

    The path to the ``rootfs.cpio.gz`` refers to <rpi3-project>, replace this so
    it matches your setup.

.. code-block:: bash

    $ cd /srv/nfs/rpi
    $ sudo gunzip -cd <rpi3-project>/out-br/images/rootfs.cpio.gz | sudo cpio -idmv
    $ sudo rm -rf /srv/nfs/rpi/boot/*

uboot.env configuration
=======================
The file ``uboot.env`` contains boot configurations that tells what binaries to
load and at what addresses. When using NFS you need to tell U-Boot where the NFS
server is located (IP and path). Since the exact IP and path varies for each
user, we must update ``uboot.env`` accordingly.

There are two ways to update ``uboot.env``, one is to update
``uboot.env.txt`` (in :ref:`build`) and the other is to update directly from
the U-Boot console. Pick the one that you suits your needs. We will cover each
of them separately here.

Change uboot.env.txt
====================
In an editor open: ``<rpi3-project>/build/rpi3/firmware/uboot.env.txt`` and
change:

    - ``nfsserverip`` to match the IP address of your NFS server.
    - ``gatewayip`` to the IP address of your router.
    - ``nfspath`` to the exported filesystem in your NFS share.

As an example a section of ``uboot.env.txt`` could look like this:

.. code-block:: c
    :emphasize-lines: 2,4,5

    # NFS/TFTP boot configuraton
    gatewayip=192.168.1.1
    netmask=255.255.255.0
    nfsserverip=192.168.1.100
    nfspath=/srv/nfs/rpi

Next, you need to re-generate ``uboot.env``:

.. code-block:: bash

    $ cd <rpi3-project>/build
    $ make u-boot-env-clean
    $ make u-boot-env

Finally, you need to copy the updated ``<rpi3-project>/out/uboot.env`` to the
**BOOT** partition of your SD-card (mount it as described in
:ref:`rpi3_build_instructions` and then just overwrite (``cp``) the file on the
**BOOT** partition of your SD-card).

Update u-boot.env from U-Boot console
=====================================
Boot up the device until you see U-Boot running and counting down, then hit any
key and will see the ``U-Boot>`` prompt. You can then update the
``nfsserverip``, ``gatewayip`` and ``nfspath`` by writing

.. code-block:: bash

    U-Boot> setenv nfsserverip '192.168.1.100'
    U-Boot> setenv gatewayip '192.168.1.1'
    U-Boot> setenv nfspath '/srv/nfs/rpi'

If you want those environment variables to persist between boots, then type.

.. code-block:: bash

    U-Boot> saveenv

Boot up with NFS
================
With all preparations above done correctly, you should now be able to boot up
the device and kernel, secure side OP-TEE and the entire root filesystem should
be loaded from the network shares (NFS). Power up the Raspberry, halt in U-Boot and
then type.

.. code-block:: bash

    U-Boot> run nfsboot


If everything works, you can simply copy paste files like ``xtest``, Trusted
Applications and other things that usually resides on the host PC's filesystem,
i.e., directly from your build folders to the ``/srv/nfs/rpi/...`` folders. By
doing so you don't have to reboot the device when doing development and testing.
Just rebuild and copy is sufficient.

.. note::

    You **cannot** make symlinks in the NFS share to the built files, i.e., you
    must copy them!


.. _rpi3_jtag:

JTAG
****
To enable JTAG you need to add a line saying ``enable_jtag_gpio=1`` in
``config.txt``. There are two ways you can do this, both requires that you to
mount the **BOOT** partition on the SD-card at your computer (see the ``make
img-help`` step under :ref:`rpi3_build_instructions`). **After** you have
mounted the BOOT partition continue with whichever way is most suitable for you.

Change config.txt directly
==========================
With your editor, open ``/media/boot/config.txt`` and add a line
``enable_jtag_gpio=1``, save the file, unmount the BOOT partition and you're
good to go after rebooting the device.

Rebuild and untar
=================
1. With your editor, open ``<rpi3-project>/build/rpi3/firmware/config.txt`` and
   add a line ``enable_jtag_gpio=1``, save the file.

2. ``$ cd <rpi3-project>/build && make``

3. ``$ cd /media``

4. ``$ sudo gunzip -cd <rpi3-project>/out-br/images/rootfs.cpio.gz | sudo cpio -idmv "boot/*"``

   .. note::

    You didn't forget to mount the BOOT partition before trying this step?

5. Unmount the BOOT partition and you're good to go after rebooting the device.


.. _rpi3_jtag_cable:

JTAG/RPi3 cable
===============
We have created our own cables that consists of a standard 20-pin JTAG
connector and a 22-pin connector for the Raspberry Pi 3 itself. Then using a
ribbon cable we have connected the cables according to the table below (JTAG pin
<-> Raspberry Pi 3 Header pin).

+----------+--------+--------+------+-----------------+
| JTAG pin | Signal | GPIO   | Mode | RPi3 Header pin |
+==========+========+========+======+=================+
| 1        | 3v3    | N/A    | N/A  | 1               |
+----------+--------+--------+------+-----------------+
| 3        | nTRST  | GPIO22 | ALT4 | 15              |
+----------+--------+--------+------+-----------------+
| 5        | TDI    | GPIO26 | ALT4 | 37              |
+----------+--------+--------+------+-----------------+
| 7        | TMS    | GPIO27 | ALT4 | 13              |
+----------+--------+--------+------+-----------------+
| 9        | TCK    | GPIO25 | ALT4 | 22              |
+----------+--------+--------+------+-----------------+
| 11       | RTCK   | GPIO23 | ALT4 | 16              |
+----------+--------+--------+------+-----------------+
| 13       | TDO    | GPIO24 | ALT4 | 18              |
+----------+--------+--------+------+-----------------+
| 18       | GND    | N/A    | N/A  | 14              |
+----------+--------+--------+------+-----------------+
| 20       | GND    | N/A    | N/A  | 20              |
+----------+--------+--------+------+-----------------+

.. warning::

    Be careful and cross check the wiring as incorrect wiring might **damage**
    your device! Also be careful to connect the cable correctly at both ends
    (don't flip it and don't put it at the wrong pins in the Raspberry Pi 3
    side).


.. _rpi3_uart_cable:

UART/RPi3 cable
***************
In addition to the JTAG connections we have also wired up the RX/TX to be able
to use the UART. Note, for this you don't need to do JTAG wirings, i.e., it's
perfectly fine to just wire up the UART only. There are many ready made cables
for this on the net (`eBay`_) and cost almost nothing. Get one of those if you
**don't** intend to use JTAG.

+-------------+-------+--------+------+----------------+
| UART pin    | Signal| GPIO   | Mode | RPi3 Header pin|
+=============+=======+========+======+================+
| Black (GND) | GND   | N/A    | N/A  | 6              |
+-------------+-------+--------+------+----------------+
| White (RXD) | TXD   | GPIO14 | ALT0 | 8              |
+-------------+-------+--------+------+----------------+
| Green (TXD) | RXD   | GPIO15 | ALT0 | 10             |
+-------------+-------+--------+------+----------------+

.. warning::

    Be careful and cross check the wiring as incorrect wiring might **damage**
    your device!


.. _rpi3_openocd:

OpenOCD
*******
Build OpenOCD
=============
Before building OpenOCD, ensure that you have the ``libusb-dev`` installed.

.. code-block:: bash

    $ sudo apt-get install libusb-1.0-0-dev

We are using the `official OpenOCD`_ release, simply clone that to your computer
and then building is like a lot of other software, i.e.,

.. code-block:: bash

    $ git clone http://repo.or.cz/openocd.git
    $ cd openocd
    $ ./bootstrap
    $ ./configure
    $ make


.. note::

    In recent versions of OpenOCD, the legacy ft2332 support has been depracted.
    All these devices now uses libftdi instead. From OpenOCD release notes:
    `"GPL-incompatible FTDI D2XX library support dropped (Presto, OpenJTAG and
    USB-Blaster I are using libftdi only now)"`.

We leave it up to the reader of this guide to decide if he wants to install it
properly (``make install``) or if he will just run it from the tree directly.
The rest of this guide will just run it from the tree.

OpenOCD RPi3 configuration file
===============================
Unfortunately, the necessary `RPi3 OpenOCD config`_ isn't upstreamed yet into
the `official OpenOCD`_ repository, so you should use the one stored here
``<rpi3-project/build/rpi3/debugger/pi3.cfg``.

Running OpenOCD
===============
Depending on the JTAG debugger you are using you'll need to find and use the
interface file for that particular debugger. We've been using `J-Link
debuggers`_ and `Bus Blaster`_ successfully. To start an OpenOCD session using a
J-Link device you type:

.. code-block:: bash

    $ cd <openocd>
    $ ./src/openocd -f ./tcl/interface/jlink.cfg -f <rpi3-project>/build/rpi3/debugger/pi3.cfg

For Bus Blaster type:

.. code-block:: bash

    $ ./src/openocd -f ./tcl/interface/ftdi/dp_busblaster.cfg \ -f <rpi3_repo_dir>/build/rpi3/debugger/pi3.cfg

To be able to write commands directly to OpenOCD, you simply open up another
shell and type:

.. code-block:: bash

    $ nc localhost 4444

From there you can set breakpoints, examine memory etc ("``> help``" will give
you a list of available commands). Having that said, if you connect to OpenOCD
using GDB, then there is not much incentive connecting to OpenOCD directly,
since you will be able to do the same in GDB by the ``monitor`` command.

Use GDB
=======
OpenOCD will by default listen to GDB connections on port ``3333``. So after
starting OpenOCD, make a connection to GDB.

.. code-block:: bash

    # Ensure that you have "gdb" in your $PATH
    $ aarch64-linux-gnu-gdb -q
    (gdb) target remote localhost:3333

To load symbols you just use the ``symbol-file <path/to/my.elf`` as usual. For
convenience you can create an alias in the ``~/.gdbinit`` file. For TEE core
debugging this works:

.. code-block:: none

    define jtag_rpi3
      target remote localhost:3333
      symbol-file <rpi3-project>/optee_os/out/arm/core/tee.elf
    end

So, when running GDB, you simply type: ``(gdb) jtag_rpi3`` and it will both
connect and load the symbols for TEE core. For Linux kernel and other binaries
you would do the same.

Debug session example
=====================
After making an initial Raspberry Pi 3 build for OP-TEE where you've enabled
JTAG, installed and built OpenOCD, connected the JTAG cable, then you're ready
for debugging OP-TEE using JTAG on Raspberry 3. Boot up the Raspberry Pi 3 until
you are in Linux and ready to run xtest. Start a new shell (on the host machine)
where you run OpenOCD:

.. code-block:: bash

    $ cd <openocd>
    $ ./src/openocd -f ./tcl/interface/jlink.cfg -f <rpi3-project>/build/rpi3/debugger/pi3.cfg

Start another shell, where you run GDB

.. code-block:: bash

    $ <rpi3-project>/toolchains/aarch64/bin/aarch64-linux-gnu-gdb -q
    (gdb) target remote localhost:3333
    (gdb) symbol-file <rpi3-project>/optee_os/out/arm/core/tee.elf

Next, try to set a breakpoint for the function ``hmac_init``, here use
**hardware** breakpoints (i.e., ``hb``)!

.. code-block:: bash

    (gdb) hb hmac_init
    Hardware assisted breakpoint 2 at 0x1012a178: file core/lib/libtomcrypt/src/mac/hmac/hmac_init.c, line 65.
    (gdb) c
    Continuing.

In the UART console (RPi3/Linux), run xtest.

.. code-block:: bash

    # xtest

And shortly thereafter you will see GDB stops on your breakpoint and from there
you can debug using normal GDB commands.


.. _`Authentication Framework`: https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/auth-framework.rst
.. _Bus Blaster: http://dangerousprototypes.com/docs/Bus_Blaster
.. _core team: https://github.com/orgs/OP-TEE/teams/linaro/members
.. _eBay: https://www.ebay.com/sch/i.html?&_nkw=UART+cable
.. _J-Link debuggers: https://www.segger.com/jlink_base.html
.. _Linaro rootfs: http://releases.linaro.org/debian/images/installer-arm64/latest/linaro*.tar.gz
.. _official OpenOCD: http://openocd.org
.. _RPi3 in TF-A: https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/plat/rpi3.rst
.. _RPi3 OpenOCD config: https://github.com/OP-TEE/build/blob/master/rpi3/debugger/pi3.cfg
.. _Sequitur Labs: http://www.sequiturlabs.com
