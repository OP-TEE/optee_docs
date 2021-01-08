.. _hikey960:

#########
HiKey 960
#########

The instructions here will tell how to run OP-TEE on `HiKey 960`_.

Supported HiKey960 boards
*************************
There are three different versions of the HiKey960 board.

+----------+--------------------+--------+-------+--------------------------------------------------+
| Name     | Manufacturer       | Memory | Flash | Comment                                          |
+==========+====================+========+=======+==================================================+
| HiKey960 | Archermind/LeMaker | 4GB    | 32GB  | v2 uses DIP Switches (SW2201), rev B has 4GB RAM |
+----------+--------------------+--------+-------+--------------------------------------------------+
| HiKey960 | Archermind/LeMaker | 3GB    | 32GB  | v2 uses DIP Switches (SW2201), rev A has 3GB RAM |
+----------+--------------------+--------+-------+--------------------------------------------------+
| HiKey960 | Archermind/LeMaker | 3GB    | 32GB  | v1 uses Jumpers (J2001)                          |
+----------+--------------------+--------+-------+--------------------------------------------------+

UART adapter board
******************
Everything is configured to use the 96Boards `UART Serial`_ adapter. The UART is
by default configured to UART6. If you have a v1 board and need to use UART5,
then you need to change that before building. See ``CFG_CONSOLE_UART`` in
`hikey960.mk`_.

Build instructions
******************
Just follow the instructions at ":ref:`get_and_build_the_solution`". If ``make
flash`` doesn't work, try ``make recovery``.

For the 4GB RAM board version (rev B), an update to the ``CFG_DRAM_SIZE_GB`` 
setting in `conf.mk`_ is needed. See the following patch:

.. code-block::

  --- ../optee_os/core/arch/arm/plat-hikey/conf.mk.orig	2021-01-08 13:09:22.889289047 +0100
  +++ ../optee_os/core/arch/arm/plat-hikey/conf.mk	2021-01-08 13:09:32.261380826 +0100
  @@ -48,7 +48,7 @@
   
   ifeq ($(PLATFORM_FLAVOR),hikey960)
   CFG_CONSOLE_UART ?= 6
  -CFG_DRAM_SIZE_GB ?= 3
  +CFG_DRAM_SIZE_GB ?= 4
   CFG_CORE_BGET_BESTFIT ?= y
   ifeq ($(CFG_ARM32_core),y)
   CFG_ASAN_SHADOW_OFFSET ?= 0x372E38E0


Recovery
********
If you manage to corrupt the device, such that fastboot doesn't load
automatically on boot, then you will need to run the recovery procedure.
Basically what you will need to do is use another make target and change some
jumpers. All that is described when you run the target:

External guide
**************
https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/plat/hikey960.rst

.. code-block:: bash

    $ make recovery

.. _HiKey 960: https://www.96boards.org/product/hikey960/
.. _hikey960.mk: https://github.com/OP-TEE/build/blob/master/hikey960.mk
.. _conf.mk: https://github.com/OP-TEE/optee_os/blob/master/core/arch/arm/plat-hikey/conf.mk
.. _UART Serial: https://www.96boards.org/product/uartserial/
