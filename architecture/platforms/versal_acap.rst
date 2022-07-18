.. _versal_acap:

#############################################
VERSAL Adaptive Compute Acceleration Platform
#############################################


.. _overview :

Overview
********

The OP-TEE support for the AMD/Xilinx Versal Adaptive Compute Acceleration Platform (`Versal ACAP`_) delegates most of its functionality to the `PLM firmware`_ executing in the MicroBlaze processor.
In the case of cryptographic operations, those operations or ciphers or keys not supported by the hardware will be routed to the `Libtomcrypt`_ software based implementation.

The services offered by PLM firmware are decided at build time: it is therefore important that the PLM firmware build configuration enables the services that OP-TEE will require.

As an example, if OP-TEE is configured to generate a hardware unique key, it  will need access to the PLM Physical Unclonable Function and NVM services.

.. note::
   Communication between OP-TEE and the PLM uses the IPI mailbox controller being the IPI used selectable via ``CFG_VERSAL_MBOX_IPI_ID``.

As described in freely available documentation for the Versal ACAP `boot-flow`_, the BootROM handles loading the PLM, while the PLM will handle loading the rest of the images including OP-TEE.

A reference BIF file supporting an OP-TEE instance capable of loading the FPGA pdi can be seen below. In this example OP-TEE should be configured with ``CFG_DT=y`` and ``CFG_DT_ADDR=0x00001000``.
If enabled the platform expects the FPGA bitstream at 0x40000000; the location is configurable using ``CFG_VERSAL_FPGA_DDR_ADDR``.

.. code-block:: none

	ROM_image:
	{
		image {
                      { type=bootimage, file=vpl_gen_fixed.pdi }
	              { type=bootloader, file=plm.elf }
	              { core=psm, file=psmfw.elf }
	        }
	        image {
	              id = 0x1c000000, name=apu_subsystem
	              { type=raw, load=0x00001000, file=versal-vck190-revA-x-ebm-01-revA.dtb }
	              { type=raw, load=0x40000000, file=fpga.pdi }
	              { core=a72-0, exception_level=el-3, trustzone, file=bl31.elf }
 	              { core=a72-0, exception_level=el-2, file=u-boot.elf }
	              { core=a72-0, exception_level=el-1, trustzone, load=0x60000000, startup=0x60000000, file=tee-raw.bin }
	         }
         }


To build the boot-able image AMD/Xilinx uses the `bootgen tool`_; this tool aggregates all the different images in a single binary.

The available configuration options depend on the architecture.

In the case of Versal ACAP using the previously mentioned BIF file, the boot.bin generation would be as follows:

.. code-block:: none
		
        $ bootgen -arch versal -image file.bif -o boot.bin
   

Cryptographic driver
********************

The Versal ACAP cryptography driver rests on the PLM's `xilsecure`_ service.
It provides hardware assisted support for:

    1. SHA3-384
    2. RSA 2048, 4096   
    3. ECC sign/verify
    4. AES-GCM   

Other drivers
*************

The ``Versal ACAP eFUSE`` driver uses the PLM `xilnvm`_ service.
Access to certain eFuses require specific PLM configuration flags not selectable at run-time.

.. note::
   It is therefore left down to the user to make sure that the PLM has been configured as expected. 

The ``Versal ACAP PUF`` driver uses the PLM `xilpuf`_ service.

At the time of this writing, the platform support includes three native drivers:

    1. ``Mailbox driver``
    2. ``TRNG driver``
    3. ``GPIO driver``

       
Hardware Unique Key
*******************

The calculation of the Hardware Unique Key - used to derive the RPMB secret - is similar to the Zynqmp platform: a digest is generated from the DNA eFUSE identifier and then GCM-AES encrypted.
The symmetric key for the AES-GCM encryption engine can however be selected at build time using the configuration option ``CFG_VERSAL_HUK_KEY``.

Contrary to what happens in the Zynqmp platform, the PUF KEK is available also on non-secured boards (i.e: boards not booting signed images).

This means that the driver has no mechanism for restricting the generation of the HUK to using data based on information ``only available`` to secured systems.

.. note::
   The security of the platform will depend on the process used to generate and lock the keys.

	
.. _boot-flow:
   https://docs.xilinx.com/r/en-US/ug1304-versal-acap-ssdg/Boot-Flow
   
.. _bootgen tool:
   https://github.com/Xilinx/bootgen

.. _Libtomcrypt:
   https://optee.readthedocs.io/en/latest/architecture/crypto.html?highlight=libtomcrypt#libtomcrypt
	
.. _PLM firmware:
   https://github.com/Xilinx/embeddedsw

.. _Versal ACAP:
   https://www.xilinx.com/products/silicon-devices/acap/versal.html

.. _xilnvm:
   https://github.com/Xilinx/embeddedsw/tree/master/lib/sw_services/xilnvm

.. _xilpuf:
   https://github.com/Xilinx/embeddedsw/tree/master/lib/sw_services/xilpuf
   
.. _xilsecure:
   https://github.com/Xilinx/embeddedsw/tree/master/lib/sw_services/xilsecure
   
