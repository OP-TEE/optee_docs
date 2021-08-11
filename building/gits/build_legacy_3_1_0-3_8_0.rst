.. _build_legacy_3_1_0-3_8_0:

######################################
Build stable releases 3.1.0 to 3.8.0
######################################
If you have a recent enough version of the Google repo tool (>= ``2.0.0``)
and follow the normal build procedure at ":ref:`get_and_build_the_solution`",
you will likely get an error during ``repo init`` with the following OP-TEE
versions and platforms:

    - 3.1.0 to 3.5.0: ``qemu_v8.xml``, ``rpi3.xml``
    - 3.6.0 to 3.8.0: ``default.xml`` (QEMU), ``qemu_v8.xml``, ``rpi3.xml``

The typical error message is:

.. code-block:: bash

    $ repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml -b 3.3.0
    [...]
    fatal: manifest 'qemu_v8.xml' not available
    fatal: <linkfile> invalid "src": ../toolchains/aarch64/bin/aarch64-linux-gnu-gdb: bad component: ..

The workaround is to checkout repo version ``1.13.9`` manually:

.. code-block:: bash

    $ repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml -b 3.3.0
    # Above error occurs, ignore it
    $ (cd .repo/repo; git checkout v1.13.9)
    $ repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml -b 3.3.0
    # Should not error out. Then proceed with 'repo sync' and build.
