.. _faq:

##########################
Frequently Asked Questions
##########################

.. contents:: Table of Contents

----

Abbreviations
*************
:OP-TEE: Open Portable TEE
:TA: Trusted Application
:TEE: Trusted Execution Environment
:TZASC: TrustZone Address Space Controller
:TZPC: TrustZone Protection Controller

----

Architecture
************
Q: Which platforms/architectures are supported?
===============================================
    - The :ref:`platforms_supported` page lists all platforms and architectures
      currently supported in the official tree.

Q: Are 32-bit as well as 64-bit support?
========================================
    - Both 32- and 64-bit are fully supported for all OP-TEE components.

Q: Does OP-TEE support mixed-mode, i.e., both AArch32 and AArch64 Trusted Applications on top of an AArch64 core?
=================================================================================================================
    - Yes!

Q: What’s the maximum size for heap and stack? Can it be changed?
=================================================================
    - Yes, it can be changed. In the current setup (for vexpress for example),
      there are ``32MB DDR`` dedicated for OP-TEE. ``1MB`` for ``TEE RAM`` and
      ``1MB`` for ``PUB RAM``, this leaves ``30MB`` for Trusted Applications. In
      the Trusted Applications, you set ``TA_STACK_SIZE`` and ``TA_DATA_SIZE``.
      Typically, we set stack to ``2KB`` and data to ``32K``. But you are free
      to adjust those according to the amount of memory you have available. If
      you need them to be bigger than ``1MB`` then you also must adjust TA’s MMU
      L1 table accordingly, since default section mapping is 1MB.

Q: What is the size of OP-TEE itself?
=====================================
    - As of 2016.01, optee_os is about ``244KB`` (release build). It is
      preferred to run :ref:`optee_os` entierly in SRAM, but if there is not
      enough room, DRAM can be used and protected with TZASC. We are also
      looking into the possibility of creating a ‘minimal’ OP-TEE, i.e. a
      limited OP-TEE usable even in a very memory constrained environment, by
      eliminating as many memory-hungry parts as possible. There is however no
      ETA for this at the moment.

    - You can check the memory usage by using the ``make mem_usage`` target in
      :ref:`optee_os`, for example:

      .. code-block:: bash

        $ make ... mem_usage
        # Which will output a file with the figures here:
        # out/arm/core/tee.mem_usage

      You will of course get different sizes depending on what compile time
      flags you have enabled when running `make mem_usage`.

Q: Can NEON optimizations be done in OP-TEE?
============================================
    - Yes (for additional information, please also see `Issue#953`_)

Q: Can I use C++ libraries in OP-TEE?
=====================================
    - C++ libraries are currently not supported. Technically, it is possible but
      will require a fair amount of work to implement, especially more so if
      exceptions are required. There are currently no plans to do this.

    - See `Issue#2628`_ for related information.

Q: Would using `malloc()` in OP-TEE give physically contiguous memory?
======================================================================
    - ``malloc()`` in OP-TEE currently gives physically contiguous memory. It is
      not guaranteed as it is not mentioned anywhere in the documentation, but
      in practice the heap only has physically contiguous memory in the pool(s).
      The heap in OP-TEE is normally quite small, ~24KiB, and could be a bit
      fragmented.

Q: Can I limit what CPUs / cores OP-TEE runs on?
================================================
    - Currently it’s up to the kernel to decide which core it runs on, i.e, it
      will be the same core as the one initiating the SMC in Linux. Please also
      see `Issue#1194`_.

Q: How is OP-TEE being scheduled?
=================================
    - OP-TEE does not have its own scheduler, instead it is being scheduled by
      Linux kernel. For more information, please see `Issue#1036` and
      `Issue#1183`_.

----

Board support
*************
Q: How do I port OP-TEE to another platform?
============================================
    - Start by reading the :ref:`porting_guidelines`.

    - See the :ref:`presentations` page. There might be some interesting
      information in the "LCU14-302 How To Port OP-TEE To Another Platform" deck
      and video. Beware that the presentation is more than five years old, so
      even though it is a good source, there might be parts that are not
      relevant any longer.

    - As a good example for
      
        - **Armv8-A** patch enabling OP-TEE support on a new device, please see
          the `ZynqMP port`_ that enabled support for running OP-TEE on `Xilinx
          UltraScale+ Zynq MPSoC`. Besides that there are similar patches for
          `Juno port`_, `Raspberry Pi3 port`_, `HiKey port`_.

        - **ARMv7-A**, please have a look at the `Freescale ls1021a port`_,
          another example would be the `TI DRA7xx port`_.

----

Building
********
Q: I got build errors running latest, why?
==========================================
    - What did you try to build? Only :ref:`optee_os`? A full OP-TEE developer
      setup using QEMU, HiKey, RPi3, Juno using repo? AOSP? OpenEmbedded? What
      we build on daily basis are the OP-TEE developer setups (see
      :ref:`optee_developer_setup`) , but other builds like AOSP and
      OpenEmbedded are builds that we try from time to time, but not very often
      within Security Working Group. Having that said there are other teams in
      Linaro working with such builds, but they most often base their builds on
      OP-TEE stable releases.

    - By running latest instead of stable also comes with a risk of getting
      build errors due to version and/or interdependency skew which can result
      in build error. Now, such issues most often affects running xtest and not
      the building. If you however clean all gits and do a ``repo sync -d``. Then
      we're almost 100% sure you will get back to a working state again, since
      as mentioned in next bullet, we build (and run xtest) on all QEMU on all
      patches sent to OP-TEE.

    - Every pull request in OP-TEE are tested on hardware (see
      :ref:`how_are_you_testing_optee`).

Q: I got build errors running stable tag x.y.z, why?
====================================================
    - Stable releases are quite well tested both in terms of building for all
      supported platforms and running xtest on all platforms, so if you cannot
      get that to build and run, then there is a great chance you have something
      wrong on your side. All platforms that has been tested on a stable release
      can be found in `CHANGELOG.md`_ file. Having that said, we do make mistakes
      on stable builds also from time to time.

Q: I get `gcc XYZ` or `g++ XYZ` compiler error messages?
========================================================
    - Most likely you're trying to build OP-TEE using the regular x86 compiler
      and not the using the Arm toolchain. Please install the
      :ref:`prerequisites` and make sure you have gotten and installed the Arm
      toolchains as described at the :ref:`toolchains` page. (for additional
      information, please see `Issue#846`_).

Q: I found this build.git, what is that?
========================================
    - :ref:`build` is a git that is used in conjunction with the
      :ref:`manifest` to create full OP-TEE developer builds. It contains
      helper makefiles that makes it easy to get OP-TEE up and running on the
      setups that are using repo.

Q: When running `make` from build.git it fails to download the toolchains?
==========================================================================
- We try to stay somewhat up to date with running recent ``GCC`` versions. But
  just like everywhere else on the net things moves around. In some cases like
  `Issue#1195`_, the URL was changed without us noticing it. If you find and fix
  such an issue, please send the fix as pull request and we will be happy to
  merge it.

Q: What is the quickest and easiest way to try OP-TEE?
======================================================
    - That would be running it on QEMU on a local PC. To do that you would need to:

        - Install the OP-TEE :ref:`prerequisites`.
        - Build for QEMU according to the instructions at :ref:`qemu_v7`.
        - And :ref:`optee_test_run_xtest`.

    - By summarizing the above, you would need to:
        .. code-block:: bash

            $ sudo apt-get install [pre-reqs]
            $ mkdir optee-qemu && cd optee-qemu
            $ repo init -u https://github.com/OP-TEE/manifest.git
            $ repo sync
            $ cd build
            $ make toolchains -j2
            $ make run
            QEMU console:         (qemu) c
            Normal world shell:   # xtest

----

Certification and security reviews
**********************************
Q: Will linaro be involved in GlobalPlatform certification/qualification?
=========================================================================
    - No we will not, mainly for two reasons. The first is that there was a
      board decision that Security WG in Linaro should not be part of
      certifications. The second reason is that most often certification is done
      using a certain software version and on a unique device. I.e., it is the
      combination software + hardware that gets certified. Since Linaro have no
      own devices in production or for sale, we cannot be part of any
      certification. This is typically something that the SoC or OEM needs to
      do.

    - But it is worth mentioning that since OP-TEE is coming from a proprietary
      TEE solution that was GlobalPlatform certified on some products in the
      past and we regularly have people from some member companies running the
      extended test suite from GlobalPlatform we know that the gap to become
      GlobalPlatform certified/qualified isn’t that big.

.. _q_has_any_test_lab_been_testing_op-tee:

Q: Has any test lab been testing OP-TEE?
========================================
    - `Applus Laboratories`_ have done some side-channel attack testing and
      fault injection testing on OP-TEE using the :ref:`hikey` device. Their
      findings and fixes can be found at the `Security Advisories`_ page at
      optee.org.

    - Riscure_ did a mini-audit of OP-TEE which generated a couple of patches
      (see `PR#2745`). The `Security Advisories`_ page at optee.org will be
      updated with more information regarding that in the future.


Q: Have there been any code audit / code review done?
=====================================================
    - Full audit? No! Not something initiated by Linaro. But there has been some
      companies that have done audits internally and they have then shared the
      result with us and where relevant, we have created patches resolving the
      issues reported to us (see :ref:`q_has_any_test_lab_been_testing_op-tee`).

    - Code review, yes! Every single patch going into OP-TEE has been reviewed
      in a pull request on GitHub. We more or less have a requirement that every
      patch going into OP-TEE shall at least have one "Reviewed-by" tag in the
      patch.

    - Third party / test lab code review, no! Again some companies have reviewed
      internally and shared the result with us, but other than that no (see
      related :ref:`q_has_any_test_lab_been_testing_op-tee`)


Contribution
************
Q: How do I contribute?
=======================
    - Please see the :ref:`contribute` page.

Q: Where can I get help?
========================
    - Please see the :ref:`contact` page.

Q: I'm new to OP-TEE but I would like to help out, what can I do?
=================================================================
    - We always need help with code reviews, feel free to review any of the open
      `OP-TEE OS Pull Requests`_. Please also note that there could be open pull
      request in the other :ref:`optee_gits` that needs reviews too.

    - We always need help answering all the questions asked at `OP-TEE OS
      Issues`_.

    - If you want to try to solve a bug, please have a look at the `OP-TEE OS
      Bugs`_ or the `OP-TEE OS Enhancements`_.

    - Documentation tends to become obsolete if not maintained on regular basis.
      We try to do our best, but we're not perfect. Please have a look at
      :ref:`optee_docs` and try to update where you find gaps.

    - Enable `repo` for the device in :ref:`manifest` and :ref:`build` (and also
      :ref:`platforms_supported`) currently not using repo.

    - If you would like to implement a bigger feature, please reach out to us
      (see :ref:`contact`) and we can discuss what is most relevant to look into
      for the moment. If you already have an idea, feel free to send the
      proposal to us.

----

Interfaces
**********
Q: Which API’s have been implemented in OP-TEE?
===============================================
    - GlobalPlatform (see :ref:`globalplatform_api` for more details).
        - GlobalPlatform's TEE Client API v1.1 specification
        - GlobalPlatform's TEE Internal Core API v1.1 specification.
        - GlobalPlatform's Secure Elements v1.0 (**now deprecated**, see ``git
          log``).
        - GlobalPlatform's Socket API v1.0 (TCP and UDP, but not TLS).

    - AOSP Keymaster_ (v3) and AOSP Gatekeeper_ (see :ref:`aosp` for more
      details).

    - `Android Verified Boot 2.0`_ (AVB 2.0)

----

Hardware and peripherals
************************
Q: Can I use my own hardware IP for crypto acceleration?
========================================================
    - Yes, OP-TEE has a Crypto Abstraction Layer (see
      :ref:`cryptographic_implementation` that was designed mainly to make it
      easy to add support for hardware crypto acceleration. There you will find
      information about the abstraction layer itself and what you need to do to
      be able to support new software/hardware “drivers” in OP-TEE.

----

License
*******
Q: Under what license is OP-TEE released?
=========================================
    - The software is mostly provided under the `BSD 2-Clause`_ license.

    - The TEE kernel driver is released under GPLv2 for obvious reasons.

    - xtest (:ref:`optee_test`) uses BSD 2-Clause for code running in secure
      world (Trusted Applications etc) and GPLv2 for code running in normal
      world (client code).

Q: GlobalPlatform click-through license
=======================================
    - Since OP-TEE is a GlobalPlatform based TEE which implements the APIs as
      specified by GlobalPlatform one has to accept, the click-through license
      which is presented when trying to download the :ref:`globalplatform_api`
      specifications before start using OP-TEE.

Q: I've modified OP-TEE by using code with non BSD 2-Clause license, will you accept it?
========================================================================================
    - That is something we deal with case by case. But as a general answer, if
      it does not contaminate the BSD 2-Clause license we will accept it. Reach
      out to us (see :ref:`contact`) and we will take it from there.

----

Promotion
*********
Q: I want to get my company logo on op-tee.org, how?
====================================================
    - If your company has done significant contributions to OP-TEE, then please
      :ref:`contact` us and we will do our best to include your company. Pay
      attention to that we will review this on regular basis and inactive
      supporting companies might be removed in the future again.

----

Security vulnerabilities
************************
Q: I have a found a security flaw in OP-TEE, how can I disclose it with you?
============================================================================
    - Please see the :ref:`Contact` page and the :ref:`disclosure_policy` page.

----

Source code
***********
Q: Where is the source code?
============================
    - It is located on GitHub under the project `OP-TEE`_ and `linaro-swg`_.

Q: Where do I download the test suite called xtest?
===================================================
    - All the source code for that can be found in the git called
      :ref:`optee_test`.

    - The :ref:`globalplatform_tests` can be purchased separately.

Q: Where is the Linux kernel TEE driver?
========================================
    - You can find both the generic TEE framework including the OP-TEE driver
      included in the official Linux kernel project since v4.12. Having that
      said, we "buffer up" pending patches on a our :ref:`linux_kernel` branch.
      I.e., that is where we keep new features being developed for OP-TEE. In
      the long run we aim to completely stop using our own branch and just send
      all patches to the official Linux kernel tree directly. But as of now we
      cannot do that.

----

Testing
*******

.. _how_are_you_testing_optee:

Q: How are you testing OP-TEE?
==============================
    - There is a test suite called xtest that tests the complete TEE-solution to
      ensure that the communication between all architectural layers is working
      as it should. The test suite also tests the majority of the GlobalPlatform
      TEE Internal Core API. It has close to 50,000 and ever increasing test
      cases, and is also extendable to include the official GlobalPlatform test
      suite (see :ref:`globalplatform_tests`).

    - Every pull request in OP-TEE are built for a multitude of different platforms
      automatically using Travis_, Shippable_ and IBART_. Please have a look
      there to see whether it failed building on the platform you're using
      before submitting any issue about build errors.

    - For more information see :ref:`optee_test`.

----

Trusted Applications
********************
Q: How do I write a Trusted Application (TA)?
=============================================
    - Have a look at the :ref:`build_trusted_applications` page as well as the
      :ref:`optee_examples` page. Those provides guidelines and examples on how
      to implement basic Trusted Applications.

    - If you want to see more advanced uses cases of Trusted Applications, then
      we encourage that you have a look at the Trusted Applications
      :ref:`optee_test`.

Q: How do I link a library into a Trusted Application?
======================================================
    - See the example in :ref:`build_trusted_applications_submk`.

    - Also see `Issue#280`_, `Issue#601`_, `Issue#901`_, `Issue#1003`_.

Q: Where should I put my compiled Trusted Application on the device?
====================================================================
    - ``/lib/optee_armtz``, that is the default location where tee-supplicant
      will look for Trusted Applications.

.. _what_is_a_psuedo_ta_and_how_do_i_write_one:

Q: What is a Psuedo TA and how do I write one?
==============================================
    - A Psuedo TA is an OP-TEE firmware service offered through the generic API
      used to invoke Trusted Applications. Pseudo TA interface and services all
      runs in TEE kernel / core context. I.e., it will have access to the same
      functions, memory and hardware etc as the TEE core itself. If we're
      talking ARMv8-A it is running in ``S-EL1``.

Q: Are Psuedo **user space** TAs supported?
===========================================
    - No!

Q: Can a static TA Open/Invoke dynamic TA?
==========================================
    - Yes, for a longer discussion see `Issue#967`_, `Issue#1085`_,
      `Issue#1132`_.

Q: How can I extend the GlobalPlatform Internal Core API?
=========================================================
    - You may develop your own “Psuedo TA”, which is part of the core (see
      :ref:`what_is_a_psuedo_ta_and_how_do_i_write_one` for more information
      about the Psuedo TA).

Q: How are Trusted Applications verified?
=========================================
    - Please see the section :ref:`core_pub_priv_keypair` in the
      :ref:`porting_guidelines`.

    - Alternatively one can also build a Trusted Application and embed its raw
      binary content into the OP-TEE firmware binary. At runtime, if invoked,
      the Trusted Application will be loaded from the OP-TEE firmware image
      instead of being fetched from the normal world and authenticated in the
      secure world (see :ref:`early_ta` for more information).

Q: Is multi-core TA supported?
==============================
    - Yes, you can have two or more TAs running simultaneously. Please see also
      `Issue#1194`_.

Q: Is multi-threading supported in a TA?
========================================
    - No, there is no such concept as ``pthreads`` or similar. I.e, you cannot
      spawn thread from a TA. If you need to run tasks in parallel, then you
      should probably look into running two TAs or more simultaneously and then
      let them communicate with each other using the ``TA2TA`` interface.

Q: How can I use or call OP-TEE from native Android (apk) applications?
=======================================================================
    - Use the `Java Native Interface`_ (JNI).
    - First get familiar with `sample_hellojni.html`_ and make sure you can run
      the sample. After that, replace the C-side Implementation with for example
      :ref:`hello_world` or one of the other examples in :ref:`optee_examples`.

      .. note::

        Note that :ref:`hello_world` and other binaries in optee_examples are built
        as executables, and have to be modified to be built as a .so shared library
        instead so that it can be loaded by the Java-side Implementation.

    - Note that ``*.apk`` apps by default have no access to the TEE driver. See
      `Issue#903`_ for details. The workaround is to disable SELinux before
      launching any ``*.apk`` app that calls into OP-TEE. The solution is to
      create/write SELinux domains/rules to allow any required access, but since
      this is not a TEE-related issue, it is left as an exercise for the users.

Q: I've heard that there is a Widevine and PlayReady TA, how do I get access?
=============================================================================
    - Those can only be shared are under WMLA and NDA/MLA with Google and
      Microsoft. Linaro can help members of Linaro to get access to those. As of
      now, we cannot share it with non-members.

.. _Issue#280: https://github.com/OP-TEE/optee_os/issues/280
.. _Issue#601: https://github.com/OP-TEE/optee_os/issues/601
.. _Issue#846: https://github.com/OP-TEE/optee_os/issues/846
.. _Issue#901: https://github.com/OP-TEE/optee_os/issues/901
.. _Issue#903: https://github.com/OP-TEE/optee_os/issues/903
.. _Issue#953: https://github.com/OP-TEE/optee_os/issues/953
.. _Issue#967: https://github.com/OP-TEE/optee_os/issues/967
.. _Issue#1003: https://github.com/OP-TEE/optee_os/issues/1003
.. _Issue#1036: https://github.com/OP-TEE/optee_os/issues/1036
.. _Issue#1085: https://github.com/OP-TEE/optee_os/issues/1085
.. _Issue#1132: https://github.com/OP-TEE/optee_os/issues/1132
.. _Issue#1183: https://github.com/OP-TEE/optee_os/issues/1183
.. _Issue#1194: https://github.com/OP-TEE/optee_os/issues/1194
.. _Issue#1195: https://github.com/OP-TEE/optee_os/issues/1195
.. _Issue#2628: https://github.com/OP-TEE/optee_os/issues/2628

.. _PR#2745: https://github.com/OP-TEE/optee_os/pull/2745

.. _Android Verified Boot 2.0: https://android.googlesource.com/platform/external/avb/+/master/README.md
.. _Applus Laboratories: http://www.appluslaboratories.com/en/
.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
.. _CHANGELOG.md: https://github.com/OP-TEE/optee_os/blob/master/CHANGELOG.md
.. _Freescale ls1021a port: https://github.com/OP-TEE/optee_os/commit/85278139a8f914dddb36808861c86a472ecb0271
.. _Gatekeeper: https://source.android.com/security/authentication/gatekeeper
.. _HiKey port: https://github.com/OP-TEE/optee_os/commit/d70e78c49fc9c63b2d37c596b7ad3cbd38f8e574
.. _IBART: https://optee.mooo.com:5000
.. _Java Native Interface: http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html
.. _Juno port: https://github.com/OP-TEE/optee_os/commit/90e7497e0480892e2c262cec64e6c47242d4db7f
.. _Keymaster: https://source.android.com/security/keystore
.. _linaro-swg: https://github.com/linaro-swg
.. _OP-TEE: https://github.com/OP-TEE
.. _OP-TEE OS Bugs: https://github.com/OP-TEE/optee_os/labels/bug
.. _OP-TEE OS Enhancements: https://github.com/OP-TEE/optee_os/labels/enhancement
.. _OP-TEE OS Issues: https://github.com/OP-TEE/optee_os/issues
.. _OP-TEE OS Pull Requests: https://github.com/OP-TEE/optee_os/pulls
.. _Raspberry Pi3 port: https://github.com/OP-TEE/optee_os/commit/66d9cacf37e6bd4b0d86e7b32e4e5edefe8decfd
.. _Riscure: https://www.riscure.com
.. _sample_hellojni.html: https://developer.android.com/ndk/samples/sample_hellojni.html
.. _Security Advisories: https://www.op-tee.org/security-advisories/
.. _Shippable: https://app.shippable.com/github/OP-TEE/optee_os/dashboard
.. _TI DRA7xx port: https://github.com/OP-TEE/optee_os/commit/9b5060cd92a19b4d114a1ce8a338b18424974037
.. _Travis: https://travis-ci.org/OP-TEE
.. _ZynqMP port: https://github.com/OP-TEE/optee_os/commit/dc57f5a0e8f3b502fc958bc64a5ec0b0f46ef11a
