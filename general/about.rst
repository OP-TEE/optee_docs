############
About OP-TEE
############
OP-TEE is a Trusted Execution Environment (TEE) designed as companion to a
non-secure Linux kernel running on Arm; Cortex-A cores using the TrustZone
technology. OP-TEE implements :ref:`tee_internal_core_api` v1.1.x which is the
API exposed to Trusted Applications and the :ref:`tee_client_api` v1.0, which is
the API describing how to communicate with a TEE. Those APIs are defined in the
:ref:`globalplatform_api` specifications.

The non-secure OS is referred to as the Rich Execution Environment (REE) in TEE
specifications. It is typically a Linux OS flavor as a GNU/Linux distribution or
the AOSP.

OP-TEE is designed primarily to rely on the Arm TrustZone technology as the
underlying hardware isolation mechanism. However, it has been structured to be
compatible with any isolation technology suitable for the TEE concept and goals,
such as running as a virtual machine or on a dedicated CPU.

The main design goals for OP-TEE are:

    - **Isolation** - the TEE provides isolation from the non-secure OS and
      protects the loaded Trusted Applications (TAs) from each other using
      underlying hardware support,

    - **Small footprint** - the TEE should remain small enough to reside in a
      reasonable amount of on-chip memory as found on Arm based systems,

    - **Portability** - the TEE aims at being easily pluggable to different
      architectures and available HW and has to support various setups such as
      multiple client OSes or multiple TEEs.


OP-TEE components
*****************
OP-TEE is divided in various components:

    - A secure privileged layer, executing at Arm secure PL-1 (v7-A) or EL-1
      (v8-A) level.
    - A set of secure user space libraries designed for Trusted Applications
      needs.
    - A Linux kernel TEE framework and driver (merged to the official tree in
      v4.12).
    - A Linux user space library designed upon the GlobalPlatform
      :ref:`tee_client_api` specifications.
    - A Linux user space supplicant daemon (tee-supplicant) responsible for
      remote services expected by the TEE OS.
    - A test suite (xtest), for doing regression testing and testing the
      consistency of the API implementations.
    - An example git containing a couple of simple host- and TA-examples.
    - And some build scripts, debugging tools to ease its integration and the
      development of Trusted Applications and secure services.

These components are available from several git repositories. The main ones are
:ref:`build`, :ref:`optee_os`, :ref:`optee_client`, :ref:`optee_test`,
:ref:`optee_examples` and the :ref:`linux_kernel`.

History
*******
OP-TEE was initially developed by ST-Ericsson (and later on by
STMicroelectronics), but this was before OP-TEE got the name "OP-TEE" and was
turned into an open source project. Back then it was a closed source and a
proprietary TEE project. In 2013, ST-Ericsson obtained GlobalPlatform’s
compliance qualification with this implementation, proving that the APIs were
behaving as expected according to the GlobalPlatform specifications.

Later on the same year (2013) Linaro was about to form Security Working Group
(SWG) and one of the initial key tasks for SWG was to work on an open source
TEE project. After talking to various TEE vendors Linaro ended up working with
STMicroelectronics TEE project. But before being able to open source it there
was a need to replace some proprietary components with open source components.
For a couple of months Linaro/SWG together with engineers from
STMicroelectronics re-wrote major parts (crypto library, secure monitor, build
system etc), cleaned up the project by enforcing :ref:`coding_standards`,
running checkpatch_ etc.

June 12 2014 was the day when OP-TEE was "born" as an open source project. At
that day the OP-TEE team pushed the `first commit
<https://github.com/OP-TEE/optee_os/commit/b01047730e77127c23a36591643eeb8bb0487d68>`_
to GitHub. A bit after this Linaro also made a `press release
<https://www.linaro.org/blog/op-tee-open-source-security-mass-market/>`_ about
this. That press release contains a bit more information. At the first year as
an open source project it was owned by STMicroelectronics but maintained by
Linaro and STMicroelectronics. In 2015 there was an ownership transfer of
OP-TEE from STMicroelectronics to Linaro and since then it has been Linaro who
is the primary owner and maintainer of the project. But for the maintenance
part, it has become a shared responsibility between Linaro, Linaro members and
other companies who are using OP-TEE.

.. _checkpatch: http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/scripts/checkpatch.pl
