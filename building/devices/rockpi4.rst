.. _rockpi4:

#########
ROCK Pi 4
#########
The instructions here will tell how to run OP-TEE on `ROCK Pi 4`_ / `ROCK 4`_ boards.

Supported ROCK Pi 4 boards
**************************
There are several versions of the ROCK Pi 4 board, each with 1GB, 2GB or 4GB
RAM options. OP-TEE has been tested and is known to work with a Rock Pi 4
Model B OP1 4GB. Other variants will likely work too.

UART
****
The console can be accessed using a USB adapter connected to the board as described
in the `serial console`_ documentation.

Build instructions
******************
Just follow the ":ref:`get_and_build_the_solution`" as described in
:ref:`build`. To flash the board you will need a USB type A to type A cable.
The ``make flash`` step will tell you how to connect the cable and use the
buttons.


.. _ROCK 4: https://wiki.radxa.com/Rock4
.. _ROCK Pi 4: https://wiki.radxa.com/RockPi4
.. _serial console: https://wiki.radxa.com/Rockpi4/dev/serial-console
