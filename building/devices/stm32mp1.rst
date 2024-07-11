.. _stm32mp1:

########
STM32MP1
########

The instructions here will tell how to run OP-TEE on one of the supported
STM32MP1 boards.

Supported boards
****************

+---------------------+--------------------+------------+-------------------------------+
| Board Name          | Manufacturer       | Boot media | Hardware Description          |
+=====================+====================+============+===============================+
| `STM32MP135F-DK`_   | STMicroelectronics | SDcard     | `Wiki STM32MP135x-DK`_        |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157A-DK1`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157x-DKx`_       |
+---------------------+                    |            |                               |
| `STM32MP157D-DK1`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-DK2`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157x-DKx`_       |
+---------------------+                    |            |                               |
| `STM32MP157F-DK2`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-EV1`_  | STMicroelectronics | SDCard (1) | `Wiki STM32MP157x-EV1`_       |
+---------------------+                    |            |                               |
| `STM32MP157F-EV1`_  |                    |            |                               |
+---------------------+--------------------+------------+-------------------------------+

(1): STM32MP157x-EV1 boards also integrate an eMMC device, a NOR flash and a
Nand flash the system can boot on. OP-TEE distribution however only supports
booting from the SDcard slot.

Build instructions
******************

Follow the instructions at ":ref:`get_and_build_the_solution`".

Configuration switch ``PLATFORM`` can be used to specify the target device
as listed in table below:

+------------------------+--------------------------------------+
| Board Name             | Build configuration directive        |
+========================+======================================+
| `STM32MP135F-DK`_      | ``PLATFORM=stm32mp1-135F_DK``        |
+------------------------+--------------------------------------+
| `STM32MP157A-DK1`_     | ``PLATFORM=stm32mp1-157A_DK1_SCMI``  |
| `STM32MP157D-DK1`_     | or ``PLATFORM=stm32mp1-157A_DK1``    |
+------------------------+--------------------------------------+
| `STM32MP157C-DK2`_     | ``PLATFORM=stm32mp1-157C_DK2_SCMI``  |
| `STM32MP157F-DK2`_     | or ``PLATFORM=stm32mp1-157C_DK2``    |
+------------------------+--------------------------------------+
| `STM32MP157C-EV1`_     | ``PLATFORM=stm32mp1-157C_EV1_SCMI``  |
| `STM32MP157F-EV1`_     | or ``PLATFORM=stm32mp1-157C_EV1``    |
+------------------------+--------------------------------------+

When the build completes, generated image file sdcard.img can be found
in the generated binary images directory ``../out/bin/`` from build
root path. The images is a GPT multipartition image you can raw copy
to the target SDcard using a tool like dd.

A usual short fecth/build/load shell sequence is like the one below:

.. code-block:: bash

  $ repo init -u https://github.com/OP-TEE/manifest.git -m stm32mp1.xml
  $ repo sync
  $ cd build
  $ make toolchains
  $ make PLATFORM=stm32mp1-157C_DK2_SCMI all
  $ dd if=../out/bin/sdcard.img of=/dev/sdX conv=fdatasync status=progress
  $ sgdisk -e /dev/sdX

Command ``sgdisk -e`` fixes the GPT backup data which location depends on
storage device effective size.

.. _STM32MP135F-DK: https://www.st.com/en/evaluation-tools/stm32mp135f-dk.html
.. _STM32MP157A-DK1: https://www.st.com/en/evaluation-tools/stm32mp157a-dk1.html
.. _STM32MP157D-DK1: https://www.st.com/en/evaluation-tools/stm32mp157d-dk1.html
.. _STM32MP157C-DK2: https://www.st.com/en/evaluation-tools/stm32mp157c-dk2.html
.. _STM32MP157F-DK2: https://www.st.com/en/evaluation-tools/stm32mp157f-dk2.html
.. _STM32MP157C-EV1: https://www.st.com/en/evaluation-tools/stm32mp157c-ev1.html
.. _STM32MP157F-EV1: https://www.st.com/en/evaluation-tools/stm32mp157f-ev1.html
.. _Wiki STM32MP135x-DK: https://wiki.st.com/stm32mpu/wiki/STM32MP135x-DK_-_hardware_description
.. _Wiki STM32MP157x-DKx: https://wiki.st.com/stm32mpu/wiki/STM32MP157x-DKx_-_hardware_description
.. _Wiki STM32MP157x-EV1: https://wiki.st.com/stm32mpu/wiki/STM32MP157x-EV1_-_hardware_description
