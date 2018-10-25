.. _hikey960:

#########
HiKey 960
#########

The instructions here will tell how to run OP-TEE on `HiKey 960`_.

Supported HiKey960 boards
*************************
There are two different versions of the HiKey960 board.

+----------+--------------------+--------+-------+-------------------------------+
| Name     | Manufacturer       | Memory | Flash | Comment                       |
+==========+====================+========+=======+===============================+
| HiKey960 | Archermind/LeMaker | 3GB    | 32GB  | v2 uses DIP Switches (SW2201) |
+----------+--------------------+--------+-------+-------------------------------+
| HiKey960 | Archermind/LeMaker | 3GB    | 32GB  | v1 uses Jumpers (J2001)       |
+----------+--------------------+--------+-------+-------------------------------+

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
.. _UART Serial: https://www.96boards.org/product/uartserial/
