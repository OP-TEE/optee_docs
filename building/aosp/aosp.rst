.. todo::

   Joakim: Technically this would belong under "gits". But I think it is more
   visible to have it directly under "AOSP" under "Build and run".

.. _aosp:

####
AOSP
####
This page contains information that tells how to get OP-TEE up and running on
HiKey devices (see :ref:`hikey`, :ref:`hikey960`) together with AOSP. The build
is based on the latest OP-TEE release and updated every quarter together with
the regular OP-TEE releases.

.. note::

    We **only** use and support this static/stable configuration. If you try
    using it with latest available AOSP, there is a risk that both OP-TEE and
    other parts are not working as expected.

As a reference, there are official instructions for HiKey builds at Google
pages, see `AOSP Hikey build instructions`_.

Prerequisites
*************

	- You should already be able to build AOSP according to the official
	  instructions. Distro should have necessary packages installed, and the repo
	  tool should be installed. Note that AOSP needs to be built with Java.
	  Also make sure that the ``mtools`` package is installed, which is needed
	  to make the hikey boot image.

	- In addition, you will need the pre-requisites necessary to build
	  optee-os.

After following the AOSP setup instructions, the following additional packages
are needed.

.. todo::

    Joakim: How much does this really differ from the "global" prerequisites.
    Could we instead refer to the global as the "main" prerequisites and then
    just list the delta here?

.. code-block:: bash

	$ sudo apt-get install android-tools-adb android-tools-fastboot autoconf \
		automake bc bison build-essential cscope curl device-tree-compiler flex \
		ftp-upload gdisk iasl libattr1-dev libc6:i386 libcap-dev libfdt-dev \
		libftdi-dev libglib2.0-dev libhidapi-dev libncurses5-dev \
		libpixman-1-dev libssl-dev libstdc++6:i386 libtool libz1:i386 make \
		mtools netcat python-crypto python-serial python-wand unzip uuid-dev \
		xdg-utils xterm xz-utils zlib1g-dev python-mako openjdk-8-jdk \
		ncurses-dev realpath android-tools-fsutils dosfstools libxml2-utils

Build instructions
******************

.. code-block:: bash

    $ git clone https://github.com/linaro-swg/optee_android_manifest
    $ cd optee_android_manifest


HiKey620 - LeMaker 8GB
    .. code-block:: bash

        $ ./sync-p.sh
        $ ./build-p.sh

HiKey620 - CircuitCo 4GB
    .. code-block:: bash

        $ ./sync-p.sh
        $ ./build.sh -v p -4g

HiKey960
    .. code-block:: bash

        $ ./sync-p-hikey960.sh
        $ ./build-p-hikey960.sh

These steps should (must) finish with no errors. In case there are errors, then
there is no need trying to flash the device.

.. warning::

    - ``--force-sync`` is used which means you might **lose your work** so save
      often, save frequent, and save accordingly, especially before running
      ``sync-p.sh`` again!

    - **Attention!** Do **NOT** use ``git clean`` with ``-x`` or ``-X`` or
      ``-e`` option in ``optee_android_manifest/``, else risk **losing all
      files** in the directory!!!

.. hint::

    You can add the ``-squashfs`` option to ``build.sh`` option to make
    ``system.img`` size smaller, but this will make ``/system`` read-only, so
    you won't be able to push files to it.

For older releases (other versions of relatively stable builds), use
below instead of ``./sync-p.sh``.

.. code-block:: bash

    $ ./wrappers/sync.sh -v p -t <hikey|hikey960> \
            -bm <name of a pinned manifest file in archive/> \
            2>&1 |tee logs/sync-p.log

E.g.
    .. code-block:: bash

        $ ./wrappers/sync.sh -v p -t hikey \
            -bm pinned-manifest-stable_yvr18.xml \
            2>&1 |tee logs/sync-p.log

Other existing files are for internal development purposes ONLY and
**NOT SUPPORTED**!

Flashing the image
******************
The instructions for flashing the image can be found in detail under
``device/linaro/hikey/installer/hikey{960}/README`` in the tree.

    1. Set jumpers/switches ``1-2`` and ``3-4``, and unset ``5-6``.
    2. Reset the board. After that, invoke:

HiKey620
    .. code-block:: bash

        $ cp -a out/target/product/hikey/*.img device/linaro/hikey/installer/hikey/
        $ sudo ./device/linaro/hikey/installer/hikey/flash-all.sh /dev/ttyUSBn

HiKey960
    .. code-block:: bash

        $ cp -a out/target/product/hikey960/*.img device/linaro/hikey/installer/hikey960/
        $ sudo ./device/linaro/hikey/installer/hikey960/flash-all.sh /dev/ttyUSBn

where the ``/dev/ttyUSBn`` device is the one that appears after rebooting with
the 3-4 jumper installed. Note that the device only remains in this recovery
mode for about 90 seconds. If you take too long to run the flash commands, it
will need to be reset again.

Partial flashing
****************
The last handful of lines in the ``flash-all.sh`` script flash various images.
After modifying and rebuilding Android, it is only necessary to flash `boot`,
`system`, `cache`, `vendor` and `userdata`. If you aren't modifying the kernel,
`boot` is not necessary, either.

Experimental prebuilts
**********************
Available at http://snapshots.linaro.org/android under ``android-hikey*``
directories.

Running xtest
*************
Do NOT try to run ``tee-supplicant`` as it has already been started
automatically as a service! Once booted to the command prompt, ``xtest`` can be
run immediately from an ``adb`` shell. For more details about running OP-TEE,
please see :ref:`optee_test_run_xtest` at :ref:`optee_test`.

.. note::

    If running from the console shell, run ``su shell,shell,inet xtest``
    instead. This is due to the console ``shell`` user not belonging to the
    ``inet`` group by default. We're looking into improving this limitation, and
    contributions are welcome!

Running VTS Gtest unit for Gatekeeper and Keymaster (Optional)
**************************************************************
On the device after going into the command prompt, run:

.. code-block:: bash

    $ su system
    $ ./data/nativetest64/VtsHalGatekeeperV1_0TargetTest/VtsHalGatekeeperV1_0TargetTest
    $ ./data/nativetest64/VtsHalKeymasterV3_0TargetTest/VtsHalKeymasterV3_0TargetTest

.. note::

    These tests need to be run as the ``system`` user.

Enable adb over USB
*******************

Boot the device. On serial console:

.. code-block:: bash

    $ su setprop sys.usb.configfs 1
    $ stop adbd
    $ start adbd

Known issues
************
Adb over USB currently doesn't work on HiKey960. As a workaround, use adb over
tcpip. See https://bugs.96boards.org/show_bug.cgi?id=502 for details on how to
connect. There are still some limitations however. E.g. running ``adb shell`` or
a second ``adb`` instance will break the current adb tcpip connection. This
might be due to unstable WiFi (there are periodic error messages like ``wlcore:
WARNING corrupted packet in RX: status: 0x1 len: 76``) or just incompleteness of
the generic HiKey960 builds under P.

.. _AOSP Hikey build instructions: https://source.android.com/source/devices.html
