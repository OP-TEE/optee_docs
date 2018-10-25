.. _build:

#####
build
#####
Why this particular git? As it turns out it's totally possible to put together
everything on your own. You can build all the individual components, os, client,
xtest, Linux kernel, TF-A, TianoCore, QEMU, Buildroot_ etc and put all the
binaries at correct locations and write your own command lines, Makefiles,
shell-scripts etc that will work nicely on the devices you are interested in. If
you know how to do that, fine, please go a head. But for newcomers it's way to
much behind the scenes to be able to setup a working environment. Also, if you
for some reason want to run something in an automated way, then you need
something else wrapping it up for you.

With this particular git **built.git** our goal is to simply to make it easy for
newcomers to get started with OP-TEE using the devices we've listed in this
document.


git location
************
https://github.com/OP-TEE/build


Why repo?
*********
We discussed alternatives, initially we started out with having a simple
shell-script, that worked to start with, but after getting more gits in use and
support for more devices it started to be difficult to maintain. In the end we
ended up choosing between repo_ from the Google AOSP project and `git
submodules`_. No matter which you choose, there will always be some person
arguing that one is better than the other. For us we decided to use repo. Not
directly for the features itself from repo, but for the ability to simply work
with different manifests containing both stable and non-stable release. Using
some tips and tricks you can also speed up setup time significantly. For day to
day work with commits, branches etc we tend to use git commands directly.


.. _root_fs:

Root filesystem
***************
The rootfs in the builds that we cover here are as small as possible and is
based on a stripped down Buildroot_ configuration adding just enough in the
rootfs such that one can:

    - Boot OP-TEE.
    - Run xtest with no regressions.
    - Easily add additional developer tools like, strace, valgrind etc.

.. note::

    As a consequence of enabling "just enough", it is likely that **non-UART**
    based enviroments won't work out of the box. I.e., if you try to boot up an
    enviroment using HDMI and connect keyboards and other devices it is likely
    that things will not work. To make them work, you probably need to rebuild
    Linux kernel with correct drivers/frameworks enabled and in addition to that
    enable binaries/daemons in Buildroot that might be necessary (user space
    tools and drivers).


How do I build using AOSP / OpenEmbedded?
*****************************************
For guides how to build AOSP, please refer to our :ref:`aosp` page. For
OpenEmbedded we have no guide ready, however there are teams in Linaro who are
building OP-TEE using OpenEmbedded. If you want to get in contact with them,
please reach out to us (see :ref:`contact`).

.. _optee_developer_setup:

Platforms supported by build.git
********************************
Below is a table showing the platforms supported by build.git. OP-TEE as such
supports many more platforms, but since quite a few of the other platforms are
maintained by people outside Linaro or are using a special setup, we encourage
you to talk to the maintainer of that platform directly if you have build
related questions etc. Please see the MAINTAINERS_ file for contact information.

.. Please keep this list sorted in alphabetic order:
.. list-table::
    :header-rows: 1

    * - Platform
      - Composite flag
      - Publicly available?

    * - `ARM Juno Board`_
      - ``PLATFORM=vexpress-juno``
      - Yes

    * - `ARM Foundation FVP`_
      - ``PLATFORM=vexpress-fvp``
      - Yes

    * - `HiKey Kirin 620`_
      - ``PLATFORM=hikey``
      - Yes

    * - `HiKey 960`_
      - ``PLATFORM=hikey-hikey960``
      - Yes

    * - `MediaTek MT8173 EVB Board`_ (deprecated)
      - ``PLATFORM=mediatek-mt8173``
      - No

    * - `Poplar`_
      - ``PLATFORM=poplar``
      - Yes

    * - `QEMU`_
      - ``PLATFORM=vexpress-qemu_virt``
      - Yes

    * - `QEMUv8`_
      - ``PLATFORM=vexpress-qemu_armv8a``
      - Yes

    * - `Raspberry Pi 3`_
      - ``PLATFORM=rpi3``
      - Yes

    * - `Texas Instruments DRA7xx`_
      - ``PLATFORM=ti-dra7xx``
      - Yes

    * - `Texas Instruments AM57xx`_
      - ``PLATFORM=ti-am57xx``
      - Yes

    * - `Texas Instruments AM43xx`_
      - ``PLATFORM=ti-am43xx``
      - Yes


Manifests
*********
.. _current_version:

Current version
===============
Here is a list of manifests for the devices currently supported in
``build.git``. With these you will get a setup containing the all necessary
software components to run OP-TEE on the chosen device. Beware that this will
run latest available on OP-TEE gits meaning that if you re-sync then you will
most likely get new commits. If you need a stable/tagged version with non-moving
gits, then please refer to the next section instead.

.. Please keep this list sorted in alphabetic order:

+----------------+------------------+----------------------+
| Target         | Manifest xml     | Device documentation |
+================+==================+======================+
| AM43xx         | ``am43xx.xml``   | :ref:`ti`            |
+----------------+------------------+----------------------+
| AM57xx         | ``am57xx.xml``   | :ref:`ti`            |
+----------------+------------------+----------------------+
| ARM Juno board | ``juno.xml``     | :ref:`juno`          |
+----------------+------------------+----------------------+
| DRA7xx         | ``dra7xx.xml``   | :ref:`ti`            |
+----------------+------------------+----------------------+
| FVP            | ``fvp.xml``      | :ref:`fvp`           |
+----------------+------------------+----------------------+
| HiKey 960      | ``hikey960.xml`` | :ref:`hikey960`      |
+----------------+------------------+----------------------+
| HiKey          | ``hikey.xml``    | :ref:`hikey`         |
+----------------+------------------+----------------------+
| Poplar Debian  | ``poplar.xml``   |                      |
+----------------+------------------+----------------------+
| QEMU           | ``default.xml``  | :ref:`qemu_v7`       |
+----------------+------------------+----------------------+
| QEMUv8         | ``qemu_v8.xml``  | :ref:`qemu_v8`       |
+----------------+------------------+----------------------+
| Raspberry Pi 3 | ``rpi3.xml``     | :ref:`rpi3`          |
+----------------+------------------+----------------------+

Stable releases
===============
Starting from OP-TEE ``v3.1`` you can check out stable releases by using the
same manifests as for current version above, but with the difference that **you
also need to specify a branch** where the name corresponds to the release
version. I.e., when we are doing releases we are creating a branch with a name
corresponding to the release version. So, let's for example say that you want to
checkout a stable OP-TEE ``v3.4`` for Raspberry Pi 3, then you do like this
instead of what is mentioned further down in section
":ref:`build_get_the_source`" (note the ``-b 3.4.0``):

.. code-block:: bash

    ...
    $ repo init -u https://github.com/OP-TEE/manifest.git -m rpi3.xml -b 3.4.0
    ...

Stable releases prior to OP-TEE v3.1 (v1.0.0 to v3.0.0)
=======================================================
Before OP-TEE ``v3.1`` we used to have separate xml-manifest files for the
stable builds. If you for some reason need an older stable release, then you can
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

.. _get_and_build_the_solution:


Get and build the solution
**************************
Below we will describe the general way of how to get the source, build the
solution and how to run xtest on the device. For device specific instructions,
please see the links in the table in the ":ref:`current_version`" section.

.. _build_prerequisites:

Step 1 - Prerequisites
======================
Install prerequisites according to the :ref:`prerequisites` page.


.. _build_install_repo:

Step 2 - Install Android repo
=============================
Note that here you don't install a huge SDK, it's simply a Python script that
you download and put in your ``$PATH``, that's it. Exactly how to "install"
repo, can be found at the Google repo_ pages, so follow those instructions
before continuing.


.. _build_get_the_source:

Step 3 - Get the source code
============================
Choose the manifest corresponding to the platform you intend to use (see the
table in section ":ref:`current_version`". For example, if you intend to use
Raspberry Pi3, then at line 3 below, ``${TARGET}.xml`` shall be ``rpi3.xml``.
The ``<optee-project>`` is whatever location where you want to store the entire
OP-TEE developer setup.

.. code-block:: bash
    :linenos:
    :emphasize-lines: 3

    $ mkdir -p <optee-project>
    $ cd <optee-project>
    $ repo init -u https://github.com/OP-TEE/manifest.git -m ${TARGET}.xml [-b ${BRANCH}]
    $ repo sync -j4 --no-clone-bundle

.. hint::

    By referencing an existing and locally saved repo forest you can save lots
    of time. We are talking about doing repo sync in 30 seconds instead of 15-30
    minutes (see the :ref:`tips_and_tricks` section for more details).


.. _build_get_toolchains:

Step 4 - Get the toolchains
===========================
In OP-TEE we're using different toolchains for different targets (depends on
ARMv7-A ARMv8-A 64/32bit solutions). In any case start by downloading the
toolchains by:

.. code-block:: bash

    $ cd <optee-project>/build
    $ make -j2 toolchains


.. _build_make:

Step 5 - Build the solution
===========================
We've configured our repo manifests, so that repo will always automatically
symlink the ``Makefile`` to the correct device specific makefile, that means
that you simply start the build by running (still in ``<optee-project>/build``)

.. code-block:: bash

    $ make -j `nproc`

This step will also take some time, but you can speed up subsequent builds by
enabling ccache_ (again see :ref:`tips_and_tricks`).

.. hint::

    **If you're having build issues**, then you can pipe the entire build log to
    a file, which makes it easier to search for the issue using a regular
    editor. In that case also avoid the ``-j`` flag so it's easier to see in what
    order things are happening. To create a ``build.log`` file do: ``$ make 2>&1
    | tee build.log``


.. _build_flash:

Step 6 - Flash the device
=========================
On **non-emulated** solutions (this means that you shouldn't do this step when
you are running QEMU-v7/v8 and FVP), you will need to flash the software in some
way. We've tried to "hide" that under the following make target:

.. code-block:: bash

    $ make flash

But, since some devices are trickier to flash than others, please see the
:ref:`device_specific`. See this just as a general instruction.

Step 7 - Boot up the device
===========================
This is device specific (see :ref:`device_specific`).


.. _build_tee_supplicant:

Step 8 - Load tee-supplicant
============================
On **most** solutions tee-supplicant is already running (check by running ``$ ps
aux | grep tee-supplicant``) on others not. If it's **not** running, then start
it by running:

.. code-block:: bash

    $ tee-supplicant -d

.. note::
    If you've built using our manifest you should not need to modprobe any
    OP-TEE/TEE kernel driver since it's built into the kernel in all our setups.


.. _build_run_xtest:

Step 9 - Run xtest
==================
The entire xtest test suite has been deployed when you we're making the builds
in previous steps, i.e, in general there is no need to copy any binaries
manually. Everything has been put into the :ref:`root_fs` automatically. So, to
run xtest, you simply type:

.. code-block:: bash

    $ xtest

If there are no regressions / issues found, xtest should end with something like
this:

.. code-block:: none
    
    ...
    +-----------------------------------------------------
    23476 subtests of which 0 failed
    67 test cases of which 0 failed
    0 test case was skipped
    TEE test application done!

.. hint::

    For other ways to run xtest, please refer to the ":ref:`optee_test_run_xtest`"
    page at :ref:`optee_test`.

.. _tips_and_tricks:

Tips and Tricks
***************
Reference existing project to speed up repo sync
================================================
Doing a ``repo init``, ``repo sync`` from scratch can take a fair amount of
time. The main reason for that is simply because of the size of some of the gits
we are using, like for the Linux kernel and EDK2. With repo you can reference an
existing forest and by doing so you can speed up repo sync to taking 30 seconds
instead of 15-30 minutes. The way to do this are as follows.

    1. Start by setup a clean forest that you will not touch, in this example,
       let us call that ``optee-ref`` and put that under for
       ``$HOME/devel/optee-ref``. This step will take somewhere between 15- to
       45 minutes, depending on your connection speed to internet.

    2. Then setup a cronjob (``crontab -e``) that does a ``repo sync`` in this
       folder particular folder once a night (that is more than enough).

    3. Now you should setup your actual tree which you are going to use as your
       working tree. The way to do this is almost the same as stated in the
       instructions above (see the ":ref:`build_get_the_source`" section) , the
       only difference is that you **also** reference the other local forest
       when running ``repo init``, like this

       .. code-block:: bash

        $ repo init -u https://github.com/OP-TEE/manifest.git --reference $HOME/devel/optee-ref

    4. The rest is the same above, but now it will only take a couple of seconds
       to clone a forest.

Normally '1' and '2' above is something you will only do once. Also if you
ignore step '2', then you will **still** get the latest from official git trees,
since repo will also check for updates that aren't at the local reference.

Use ccache
==========
ccache_ is a tool that caches build object-files etc locally on the disc and can
speed up build time significantly in subsequent builds. On Debian-based systems
(Ubuntu, Mint etc) you simply install it by running:

.. code-block:: bash

    $ sudo apt-get install ccache

The makefiles in build.git are configured to automatically find and use ccache
if ccache is installed on your system, so other than having it installed you
don't have to think about anything.

.. _Buildroot: https://buildroot.org
.. _ccache: https://ccache.samba.org
.. _git submodules: https://git-scm.com/book/en/v2/Git-Tools-Submodules
.. _MAINTAINERS: https://github.com/OP-TEE/optee_os/blob/master/MAINTAINERS
.. _repo: https://source.android.com/source/downloading.html

.. Links to devices etc:
.. _ARM Juno Board: http://www.arm.com/products/tools/development-boards/versatile-express/juno-arm-development-platform.php
.. _ARM Foundation FVP: http://www.arm.com/fvp
.. _HiKey Kirin 620: https://www.96boards.org/products/hikey
.. _HiKey 960: https://www.96boards.org/product/hikey960
.. _MediaTek MT8173 EVB Board: http://www.mediatek.com/en/products/mobile-communications/tablet/mt8173
.. _Poplar: https://www.96boards.org/product/poplar/
.. _QEMU: http://wiki.qemu.org/Main_Page
.. _QEMUv8: http://wiki.qemu.org/Main_Page
.. _Raspberry Pi 3: https://www.raspberrypi.org/products/raspberry-pi-3-model-b
.. _Texas Instruments DRA7xx: http://www.ti.com/product/DRA746
.. _Texas Instruments AM57xx: http://www.ti.com/product/AM5728
.. _Texas Instruments AM43xx: http://www.ti.com/product/AM4379
