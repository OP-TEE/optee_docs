.. _build_legacy:

######################################
Build stable releases v1.0.0 to v3.0.0
######################################
Before OP-TEE ``v3.1.0`` we used to have separate xml-manifest files for the
stable builds.
If you for some reason need such an older stable release, then you can
use the ``xyz_stable.xml`` file corresponding to your device. The way to init
``repo`` is almost the same as described above, the major difference is the name
of manifest being referenced (``-m xyz_stable.xml``) and that we are referring
to a tag instead of a branch (``-b refs/tags/MAJOR.MINOR.PATCH``). So as an
example, if you need to setup the ``2.1.0`` stable release for HiKey, then you
would do like this instead of what is mentioned further down in section
":ref:`build_get_the_source`".

.. code-block:: bash

    ...
    repo init -u https://github.com/OP-TEE/manifest.git -m hikey_stable.xml -b refs/tags/2.1.0
    ...

Here is a list of targets and the names of the stable manifests files which were
supported by older releases:

.. Please keep this list sorted in alphabetic order:

+----------------+-----------------------------+
| Target         | Stable manifest xml         |
+================+=============================+
| AM43xx         | ``am43xx_stable.xml``       |
+----------------+-----------------------------+
| AM57xx         | ``am57xx_stable.xml``       |
+----------------+-----------------------------+
| ARM Juno board | ``juno_stable.xml``         |
+----------------+-----------------------------+
| DRA7xx         | ``dra7xx_stable.xml``       |
+----------------+-----------------------------+
| FVP            | ``fvp_stable.xml``          |
+----------------+-----------------------------+
| HiKey 960      | ``hikey960_stable.xml``     |
+----------------+-----------------------------+
| HiKey Debian   | ``hikey_debian_stable.xml`` |
+----------------+-----------------------------+
| HiKey          | ``hikey_stable.xml``        |
+----------------+-----------------------------+
| MTK8173        | ``mt8173-evb_stable.xml``   |
+----------------+-----------------------------+
| QEMU           | ``default_stable.xml``      |
+----------------+-----------------------------+
| QEMUv8         | ``qemu_v8_stable.xml``      |
+----------------+-----------------------------+
| Raspberry Pi 3 | ``rpi3_stable.xml``         |
+----------------+-----------------------------+

