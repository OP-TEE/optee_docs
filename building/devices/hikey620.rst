.. _hikey:

#########
HiKey 620
#########
The instructions here will tell how to run OP-TEE on `HiKey 620`_.

Multiple sources for HiKey and OP-TEE instructions?
***************************************************
First you must understand that the HiKey project as such is led by the 96Boards
project. So, if you **aren't** interested in running OP-TEE on the device, then
you should stop reading here and instead have a look at the `official HiKey
documentation`_.

For OP-TEE using HiKey you will still find information in more than one place.
There are a couple of reasons for that.

* **96Boards**: The official 96Boards project used to host some OP-TEE
  instructions and they include OP-TEE in their official releases.

* **Google**: has an `AOSP HiKey branch`_, where OP-TEE is supported to some
  extent.

* **Linaro-SWG**: The OP-TEE team has done some work related to AOSP (see the
  :ref:`AOSP` page) and there HiKey has been one of the devices in use.

If you have questions regarding the configurations above, please reach out to
the people on the right forum (96Boards, Google and Linaro-SWG).

This particular guide is maintained by the OP-TEE `core team`_ and this is what
we use when we are doing are stable releases for our OP-TEE developer builds.
I.e, for OP-TEE this should be considered as a well maintained guide with a
fully working setup.

Supported HiKey boards
**********************
There are four different versions of the HiKey board.

    +-------+--------------+--------+-------+-------------------+
    | Name  | Manufacturer | Memory | Flash | Comment           |
    +=======+==============+========+=======+===================+
    | HiKey | CircuitCo    | 1GB    | 4GB   | Green solder mask |
    +-------+--------------+--------+-------+-------------------+
    | HiKey | LeMaker      | 1GB    | 8GB   | Black solder mask |
    +-------+--------------+--------+-------+-------------------+
    | HiKey | LeMaker      | 2GB    | 8GB   | Black solder mask |
    +-------+--------------+--------+-------+-------------------+

All of them works, but where differences apply we have default configurations
that works for the LeMaker 8GB eMMC versions.

UART adapter board
******************
Everything is configured to use the `96Boards UART Adapter Board`_. The UART is
by default configured to ``UART3``. If you don't have any UART adapter board and
instead would like to use ``UART0``, then you need to change that before
building. See ``CFG_NW_CONSOLE_UART`` and ``CFG_NW_CONSOLE_UART`` in
`hikey.mk`_.

Build instructions
******************
Just follow the ":ref:`get_and_build_the_solution`" as described in
:ref:`build`. The ``make flash`` step will tell you how you should set the
jumpers on the board.

Recovery
********
If you manage to corrupt the device, so that fastboot doesn't load automatically
on boot, then you will need to run the recovery procedure. Basically what you
will need to do is use another make target and change some jumpers. All that is
described when you run the target:

.. code-block:: bash

    $ make recovery

.. _96Boards UART Adapter Board: http://www.96boards.org/product/uarts
.. _AOSP HiKey branch: https://source.android.com/setup/build/devices#620hikey
.. _core team: https://github.com/orgs/OP-TEE/teams/linaro/members
.. _HiKey 620: https://www.96boards.org/product/hikey/
.. _hikey.mk: https://github.com/OP-TEE/build/blob/master/hikey.mk
.. _official HiKey documentation: http://www.96boards.org/documentation/ConsumerEdition/HiKey/README.md
