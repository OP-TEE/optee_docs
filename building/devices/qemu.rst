On this page you will find device specific information for QEMU v7 (Armv7-A) and
QEMU v8 (Armv8-A).

.. _qemu_v7:

#######
QEMU v7
#######
The instructions here will tell how to run OP-TEE using QEMU for Armv7-A.

Build instructions
******************
As long as you pick the v7 manifest, i.e.,  ``default.xml`` the
":ref:`get_and_build_the_solution`" tells all you need to know to build and boot
up QEMU v7.


Consoles
********
After running ``make run`` you will end up in the QEMU console and it will also
spawn two UART consoles. One console containing the UART for secure world and
one console containing the UART for normal world. You will see that it stops
waiting for input on the QEMU console. To continue, do:

.. code-block:: none

    (qemu) c

Host-Guest folder sharing
*************************
You can use the VirtFS QEMU feature to avoid changing rootfs CPIO archive every
time you need to add additional files or modify existing files. To do this, you
share a folder between the guest and host operating systems. To enable and use
this feature you have to provide additional arguments when running make,
example:

.. code-block:: bash

    $ make QEMU_VIRTFS_ENABLE=y QEMU_USERNET_ENABLE=y

.. hint::

    You can also add ``QEMU_VIRTFS_HOST_DIR=<share>`` in case you don't want to
    use the default sharing location (which is the root of <qemu-v7-project>).

When QEMU with OP-TEE is up and running, you can mount the host folder in QEMU
(normal world UART).

.. code-block:: none

    # mount -t 9p -o trans=virtio host <mount_point>

``<mount_point>`` here is folder in the QEMU where you want to mount the host
PC's shared folder. So if you want to mount it at ``/mnt/host`` you typically do
this from QEMU NW/UART.

.. code-block:: none

    # mkdir -p /mnt/host
    # mount -t 9p -o trans=virtio host /mnt/host

Networking
**********
After booting QEMU, ``eth0`` will automatically receive an IP address from
QEMU via DHCP using the SLiRP user networking feature. QEMU will act as a
gateway to the host network `SLiRP`_.

Please note that ICMP won't work in the guest unless additional configuration is
made, so the ``ping`` utility won't work.

GDB - Normal world
******************
If you need to debug a client application, using GDB in a remote debugging
configuration may be useful. Remote debugging means ``gdb`` runs on your PC,
where it can access the source code, while the program being debugged runs on
the remote system (in this case, in the QEMU environment in normal world). Here
is how to do that. On your PC, build with ``GDBSERVER=y``:

.. code-block:: bash

    $ cd <qemu-v7-project>/build
    # You **only** need to rm -rf the first time you build with the new flag.
    # If you omit doing so, it's likely that you will see "stamp" errors in the
    # build log.
    $ rm -rf <qemu-v7-project>/out-br
    $ make -j8 run GDBSERVER=y

Boot up as usual

.. code-block:: bash

        (qemu) c

Inside QEMU (Normal World UART), run your application with gdbserver (for
example ``xtest 4002``):

.. code-block:: none

    # gdbserver :12345 xtest 4002
    Process xtest created; pid = 654
    Listening on port 12345

Back on your PC, open another terminal, start GDB and connect to the target:

.. code-block:: bash

    $ <qemu-v7-project>/out-br/host/bin/arm-buildroot-linux-gnueabihf-gdb
    (gdb) set sysroot <qemu-v7-project>/out-br/host/arm-buildroot-linux-gnueabihf/sysroot
    (gdb) target remote :12345

Now GDB is connected to the remote application. You may use GDB normally.

.. code-block:: none

    (gdb) b main
    (gdb) c


.. _qemu_v8:

#######
QEMU v8
#######
The instructions here will tell how to run OP-TEE using QEMU for Armv7-A.

Build instructions
******************
As long as you pick the v8 manifest, i.e.,  ``qemu_v8.xml`` the
":ref:`get_and_build_the_solution`" tells all you need to know to build and boot
up QEMU v8.

All other things (networking, GDB etc) in the v7 section above is also
applicable on QEMU v8 as long as you replace ``<qemu-v7-project>`` with
``<qemu-v8-project>`` to get the correct paths relative to your QEMU v8 setup.

.. _SLiRP: https://wiki.qemu.org/Documentation/Networking#User_Networking_.28SLIRP.29
