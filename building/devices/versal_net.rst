.. _versal_net:

###############################
AMD-Xilinx Versal Net VNX B2197
###############################

Instructions below show how to run OP-TEE on the Versal Net VNX B2197 development board.
Details of the Versal Net can be found in the Versal Net Technical Reference Manual.

Prerequisites
*************

This port requires proprietary binaries available upon request to AMD/Xilinx:

    - versal-net-bsp folder with:
        - ksb_castle_peak_b0_optee_20240507.pdi

Configuring and building for Versal Net
***************************************

Fetching source code
====================

The default build is mostly automated and follow generic OP-TEE build procedure:

.. code-block:: console

	$ mkdir ~/optee-project
	$ cd ~/optee-project
	$ repo init -u https://github.com/OP-TEE/manifest.git -m versal_net.xml
	$ repo sync -j4 --no-clone-bundle

Adding AMD-Xilinx specific components
=====================================

Copy the versal-net-bsp folder to the ``optee-project`` directory:

.. code-block:: console

	$ cp -a <path-to>/versal-net-bsp .

Building the bootimage
======================

Let's prepare the toolchains:

.. code-block:: console

	$ cd build
	$ make -j8 toolchains

At this point we have a working directory ``~/optee-project`` with all the required toolchains.

.. code-block:: console

	$ make -j8
	$ make -j8 bootimage

A JTAG bootable image is now available at ``versal/BOOT.BIN``.

Booting the image
*****************

The ADKv2 debug module exposes 4 USB interfaces to the Linux host:

    - The third one, usually ``/dev/ttyUSB2`` is used by U-Boot and Linux for their console
    - The fourth one, usually ``/dev/ttyUSB3`` displays PLM, TF-A and OP-TEE traces
      (OP-TEE traces will be moved to the other UART in a future updage)

JTAG Boot to U-Boot
===================

.. note::
   This section assumes that PetaLinux 2023.2 tools such as ``hw_server`` and ``xsdb`` are
   available in the ``PATH``. They can be downloaded and `installed`_ from the AMD-Xilinx
   website (`Downloads`_).
   
   The user these executables are run with should also have the correct UNIX access rights
   to open the underlying USB device nodes. Most of the time adding said user to the
   ``dialout`` UNIX group is enough on Ubuntu/Debian-based systems. Otherwise, run
   ``hw_server`` as root (see below).

To run the bootable image ``BOOT.BIN`` via JTAG, configure the boot switches for JTAG boot
then power up the board.

In one terminal; start ``hw_server``:

.. code-block:: console

	$ sudo hw_server

Then in another terminal, run the following commands:

.. code-block:: console

	$ xsdb
	rlwrap: warning: your $TERM is 'xterm-256color' but rlwrap couldn't find it in the terminfo database. Expect some problems.
	
	****** System Debugger (XSDB) v2023.2
	  **** Build date : Oct 10 2023-17:54:17
	    ** Copyright 1986-2022 Xilinx, Inc. All Rights Reserved.
	    ** Copyright 2022-2023 Advanced Micro Devices, Inc. All Rights Reserved.

	
	xsdb% connect                                                                                        
	tcfchan#0
	xsdb% device program BOOT.BIN

It will download and execute the image on the Versal Net platform.

Booting Linux and running tests
===============================

To properly boot Linux with the current configuration, stop automatic boot by pressing the spacebar to get to
U-Boot prompt, the run the following command:

.. code-block:: console

	U-Boot 2023.01 (Jan 23 2024 - 10:26:16 +0100)
	 
	Model: Xilinx Versal Net VNX
	DRAM:  2 GiB (effective 32 GiB)
	EL Level:EL2
	Core:  40 devices, 23 uclasses, devicetree: board
	MMC:   mmc@f1050000: 1
	Loading Environment from nowhere... OK
	In:    serial@f1930000
	Out:   serial@f1930000
	Err:   serial@f1930000
	Bootmode: JTAG_MODE
	Timeout waiting MAC address publication.
	Net:   
	ZYNQ GEM: f19f0000, mdio bus f19f0000, phyaddr 4, interface rmii
	
	Warning: ethernet@f19f0000 (eth0) using random MAC address - aa:f7:8b:a9:3e:1b
	eth0: ethernet@f19f0000
	Autoboot in 5 seconds
	(press space bar to interrupt)
	Versal NET> booti 0x27200000 0x40000000 0x27100000

When Linux has completed its boot sequence, you can login as ``root`` without any password. All
OP-TEE services should have been started at this point and you run the ``xtest`` tool to run OP-TEE tests:

.. code-block:: console

	OP-TEE embedded distrib for versal-net-vnx-b2197-revA
	buildroot login: root
	# xtest
	[...]
	regression_4005.7 FAILED first error at regression_4000.c:605
	regression_4005 FAILED
	[...]
	+----------------------------------------
	30259 subtests of which 2 failed
	133 test cases of which 1 failed
	0 test cases were skipped
	TEE test application done!

.. note::
	The ``regression 4005.7`` test failure is a known documented failure. The Versal Net
	hardware only accepts word-aligned AAD chunks of data.

Features
********

Crypto
======

The Versal Net OP-TEE port supports the following hardware-backed algorithms:

    - ECDSA Key Generation, Signature and Verification for NIST P-256, P-384 and P-521 curves
        - This is implemented through a dedicated PKI hardware engine
        - Key generation for these algorithms makes use of a dedicated hardware TRNG
    - AES-GCM for 128 and 256-bit keys
    - RSA 2048, 3072 and 4096
    - SHA3-384
    - GMAC
    - HMAC
    - TRNG
    - Other algorithms make use of ARMv8 Crypto Extensions where applicable

FPGA Loader
===========

The Versal Net OP-TEE port includes an FPGA loader pseudo-TA that can be used to load bitsreams into the PL:

.. code-block:: c
	:caption: Sample code to load a bitsream from Linux

	#define PTA_VERSAL_FPGA_UUID { 0xa6b493c0, 0xe100, 0x4a13, \
		{ 0x9b, 0x00, 0xbc, 0xe4, 0x2d, 0x53, 0xce, 0xd8 } }

	/**
	* Write FPGA bitstream
	*
	* [in]		memref[0].buffer	FPGA bitstream buffer
	* [in]		memref[0].size		FPGA bitstream buffer size
	*
	* Return codes:
	* TEE_SUCCESS - Invoke command success
	* TEE_ERROR_BAD_PARAMETERS - Incorrect input param
	* TEE_ERROR_OUT_OF_MEMORY - Could not alloc internal buffer
	* TEE_ERROR_GENERIC - PLM failure
	*/
	#define PTA_VERSAL_FPGA_WRITE		0x0

	TEE_Result load_bitsream(uint8_t *bistream, size_t size)
	{
		TEEC_Context ctx;
		TEEC_Session sess;
		TEEC_Operation op;
		TEEC_UUID uuid = PTA_VERSAL_FPGA_UUID;
		TEE_Result ret = TEE_SUCCESS;
		uint32_t origin;

		ret = TEEC_InitializeContext(NULL, &ctx);
		if (ret != TEEC_SUCCESS)
			return ret;

		/* Open a session with the TA */
		ret = TEEC_OpenSession(&ctx, &sess, &uuid,
			       TEEC_LOGIN_PUBLIC, NULL, NULL, &origin);
		if (ret != TEEC_SUCCESS)
			goto out;

		memset(&op, 0, sizeof(op));
		op.paramTypes = TEEC_PARAM_TYPES(TEEC_MEMREF_TEMP_INPUT,
						 TEEC_NONE, TEEC_NONE, TEEC_NONE);

		op.params[0].tmpref.buffer = bitstream;
		op.params[0].tmpref.size = size;

		ret = TEEC_InvokeCommand(&sess, PTA_VERSAL_FPGA_WRITE,
					 &op, &origin);

		TEEC_CloseSession(&sess);
	out:
		TEEC_FinalizeContext(&ctx);
		return ret;
	}

The example above shows a ``load_bitsream()`` function that expects a buffer
(and its size) as a parameter. This buffer holds the actual bitsream binary
loaded from Linux filesystem for instance.

.. note::
	Bitsreams loaded through this means have their size limited by
	the amount of shared memory available to OP-TEE. Bigger bitsreams
	should be loaded at boot time.

NVM
===

The Versal Net OP-TEE provides eFuses read and write APIs to other OP-TEE
components. The API is available in ``core/include/drivers/versal_nvm.h``.

.. code-block:: c
	:caption: Example - Read the DNA value

	#include <drivers/versal_nvm.h>

	TEE_Result read_dna(uint32_t *dna)
	{
		return versal_efuse_read_dna(dna, EFUSE_DNA_LEN);
	}

.. code-block:: c
	:caption: Example - Write Black Obfuscation IV

	#include <drivers/versal_nvm.h>

	TEE_Result write_black_iv(uint32_t *iv)
	{
		struct versal_efuse_ivs ivs = { };

		ivs.prgm_blk_obfus_iv = 1;
		memcpy(ivs.blk_obfus_iv, iv, EFUSE_IV_LEN);

		return versal_efuse_write_iv(&ivs);
	}

PUF
===

The Versal Net Physically Unclonable Function is support on the OP-TEE port.
The API is available in ``core/include/drivers/versal_puf.h``.

.. code-block:: c
	:caption: Example - PUF Registration

	#include <drivers/versal_puf.h>

	TEE_Result register_puf(struct versal_puf_data *data)
	{
		struct versal_puf_cfg cfg = { };

		cfg.puf_operation = VERSAL_PUF_REGISTRATION;
		cfg.shutter_value = VERSAL_PUF_SHUTTER_VALUE;
		cfg.global_var_filter = VERSAL_PUF_GLBL_VAR_FLTR_OPTION;
		cfg.read_option = VERSAL_PUF_READ_FROM_RAM;

		return versal_puf_register(data, &cfg);
	}

Testing
*******

GPIO
====

[Tracked by requirement R-4]

.. note::
	Fully testing this requires plugging an actual load on the corresponding GPIO pin.

	The pin used can be modified in ``core/pta/versal/test_pta.c`` by changing ``GPIO_TEST_PIN_ID``.

This test is available in the ``versal`` testsuite in ``xtest``:

.. code-block:: console

	# xtest -t versal 1000
	Test ID: 1000
	Run test suite with level=0
	
	TEE test application started over default TEE instance
	######################################################
	#
	# versal
	#
	######################################################

	* versal_1000 Versal Test GPIO
	o versal_1000.1 Versal PMC GPIO test
	  versal_1000.1 OK
	o versal_1000.2 Versal PS GPIO test
	  versal_1000.2 OK
	  versal_1000 OK
	+-----------------------------------------------------
	Result of testsuite versal filtered by "1000":
	versal_1000 OK
	+-----------------------------------------------------
	3 subtests of which 0 failed
	1 test case of which 0 failed
	3 test cases were skipped
	TEE test application done!

NVM
===

[Tracked by requirement R-8]

.. note::
	This test requires the ``xilnvm`` service to be enabled in the PLM.


This test is available in the ``versal`` testsuite in ``xtest``:

.. code-block:: console

	# xtest -t versal 1010
	Test ID: 1010
	Run test suite with level=0

	TEE test application started over default TEE instance
	######################################################
	#
	# versal
	#
	######################################################

	* versal_1010 Versal Test NVM
	o versal_1010.1 Versal NVM test
	  versal_1010.1 OK
	  versal_1010 OK
	+-----------------------------------------------------
	Result of testsuite versal filtered by "1010":
	versal_1010 OK
	+-----------------------------------------------------
	2 subtests of which 0 failed
	1 test case of which 0 failed
	3 test cases were skipped
	TEE test application done!

PUF
===

[Tracked by requirement R-9]

.. note::
	This test requires the ``xilnvm`` and ``xilpuf`` services to be enabled
	in the PLM.

This test is available in the ``versal`` testsuite in ``xtest``:

.. code-block:: console

	# xtest -t versal 1020
	Test ID: 1020
	Run test suite with level=0

	TEE test application started over default TEE instance
	######################################################
	#
	# versal
	#
	######################################################

	* versal_1020 Versal Test PUF
	o versal_1020.1 Versal PUF test
	  versal_1020.1 OK
	  versal_1020 OK
	+-----------------------------------------------------
	Result of testsuite versal filtered by "1020":
	versal_1020 OK
	+-----------------------------------------------------
	2 subtests of which 0 failed
	1 test case of which 0 failed
	3 test cases were skipped
	TEE test application done!

HUK
===

The Versal Hardware Unique Key driver can use several AES-GCM key sources
to derive the HUK:

	- eFUSE USR 0
	- eFUSE USR 1
	- PUF KEK
	- AES User Key 0

Which source is used can be configured in ``core/arch/arm/plat-versal/conf.mk``:

.. code-block:: makefile

	CFG_VERSAL_DUMMY_DNA ?= y
	CFG_VERSAL_HUK ?= y
	# AES-GCM supported key sources for HUK:
	#     6  : eFUSE USR 0
	#     7  : eFuse USR 1
	#    11  : PUF KEK
	#    12  : AES User Key 0 (devel)
	CFG_VERSAL_HUK_KEY ?= 12
	ifneq ($(CFG_VERSAL_HUK_KEY),$(filter 6 7 11 12,$(firstword $(CFG_VERSAL_HUK_KEY))))
	$(error Invalid value: CFG_VERSAL_HUK_KEY=$(CFG_VERSAL_HUK_KEY))
	endif

RPMB
====

[Tracked by requirement R-11]

.. warning::
   RPMB support is disabled by default because writing the RPMB key is an irreversible operation.
   To enable it, please modify the Versal configuration file (``core/arch/arm/plat-versal/conf.mk``)
   with the following patch.

.. code-block:: diff

	--- a/core/arch/arm/plat-versal/conf.mk
	+++ b/core/arch/arm/plat-versal/conf.mk
	@@ -42,7 +42,7 @@ else
	 $(call force,CFG_ARM32_core,y)
	 endif
	 
	-CFG_RPMB_FS ?= n
	+CFG_RPMB_FS ?= y
	 CFG_RPMB_TESTKEY ?= y
	 CFG_RPMB_WRITE_KEY ?=y
	 

.. note::
   This patch enables RPMB support in OP-TEE and makes it use a hardcoded development key.
   To use the hardware-bound key, the ``CFG_RPMB_TESTKEY`` configuration option must be disabled
   and the NVM service must be enabled in the PLM.

RPMB support can be verified with OP-TEE debug logs enabled:

.. code-block:: bash

	D/TC:?? 0 tee_rpmb_init:1114 RPMB: Syncing device information
	D/TC:?? 0 tee_rpmb_init:1122 RPMB: RPMB size is 2*128 KB
	D/TC:?? 0 tee_rpmb_init:1123 RPMB: Reliable Write Sector Count is 1
	D/TC:?? 0 tee_rpmb_init:1150 RPMB INIT: Deriving key
	D/TC:?? 0 tee_rpmb_key_gen:302 RPMB: Using test key
	D/TC:?? 0 tee_rpmb_init:1165 RPMB INIT: Verifying Key
	E/TC:?? 0 tee_rpmb_verify_key_sync_counter:1013 Verify key returning 0xffff0008
	D/TC:?? 0 tee_rpmb_init:1173 RPMB INIT: Auth key not yet written
	D/TC:?? 0 tee_rpmb_write_and_verify_key:1075 RPMB INIT: Writing Key value:
	D/TC:?? 0 tee_rpmb_write_and_verify_key:1076 00000000222abd30  d3 eb 3e c3 6e 33 4c 9f  98 8c e2 c0 b8 59 54 61  
	D/TC:?? 0 tee_rpmb_write_and_verify_key:1076 00000000222abd40  0d 2b cf 86 64 84 4d f2  ab 56 e6 c6 1b b7 01 e4 
	D/TC:?? 0 tee_rpmb_write_and_verify_key:1080 RPMB INIT: Verifying Key

PKI
===

.. note::
	ECDSA keypair pairwise consistency tests are enabled by default. They can be
	disabled in ``core/arch/arm/plat-versal/conf.mk`` by setting the ``CFG_VERSAL_PKI_PWCT``
	configuration option to ``n``.

PKI engine tests can be run from the command line with ``xtest``:

.. code-block:: console

	# xtest -t versal 1040
	Test ID: 1040
	Run test suite with level=0
	
	TEE test application started over default TEE instance
	######################################################
	#
	# versal
	#
	######################################################

	* versal_1040 Versal Test PKI
	o versal_1040.1 Versal PKI - Sign/Verify P256
	  versal_1040.1 OK
	o versal_1040.2 Versal PKI - Sign/Verify P384
	  versal_1040.2 OK
	o versal_1040.3 Versal PKI - Sign/Verify P521
	  versal_1040.3 OK
	  versal_1040 OK
	+-----------------------------------------------------
	Result of testsuite versal filtered by "1040":
	versal_1040 OK
	+-----------------------------------------------------
	4 subtests of which 0 failed
	1 test case of which 0 failed
	3 test cases were skipped
	TEE test application done!

Benchmarks
----------

PKI engine benchmarks can be run with the following command lines:

	- Server-side benchmarks:

	.. code-block:: console

		# xtest -t versal 1050

	- Client-side benchmarks:

	.. code-block:: console

		# xtest -t versal 1060


.. _Downloads: https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools/2023-2.html

.. _installed: https://docs.xilinx.com/r/en-US/ug1144-petalinux-tools-reference-guide/Installing-the-PetaLinux-Tool
