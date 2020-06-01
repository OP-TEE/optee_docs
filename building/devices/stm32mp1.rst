.. _stm32mp1:

########
STM32MP1
########

The instructions here will tell how to run OP-TEE on one of the supported
STM32MP1 boards.

Supported boards
****************

+---------------------+--------------------+------------+-------------------------------+
| Baord Name          | Manufacturer       | Boot media | Hardware Description          |
+=====================+====================+============+===============================+
| `STM32MP157A-DK1`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157X-DKX`_       |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-DK2`_  | STMicroelectronics | SDcard     | `Wiki STM32MP157X-DKX`_       |
+---------------------+--------------------+------------+-------------------------------+
| `STM32MP157C-EV1`_  | STMicroelectronics | SDCard     | `Wiki STM32MP157C-EV1`_       |
+---------------------+--------------------+------------+-------------------------------+

Build instructions
******************

Follow the instructions at ":ref:`get_and_build_the_solution`".

Configuration switch ``PLATFORM`` can be used to specify the target device
as listed in table below:

+------------------------+--------------------------------------+
| Board Name             | Build configuration directive        |
+========================+======================================+
| STM32MP157A-DK1        | ``PLATFORM=stm32mp1-157A_DK1``       |
+------------------------+--------------------------------------+
| STM32MP157C-DK2        | ``PLATFORM=stm32mp1-157C_DK2``       |
+------------------------+--------------------------------------+
| STM32MP157C-EV1        | ``PLATFORM=stm32mp1-157C_EV1``       |
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
  $ make PLATFORM=stm32mp1-157C_DK2 all
  $ sudo dd if=../out/bin/sdcard.img of=/dev/sdX conv=fdatasync status=progress

.. _STM32MP157A-DK1: https://www.st.com/en/evaluation-tools/stm32mp157a-dk1.html
.. _STM32MP157C-DK2: https://www.st.com/en/evaluation-tools/stm32mp157c-dk2.html
.. _STM32MP157C-EV1: https://www.st.com/en/evaluation-tools/stm32mp157c-ev1.html
.. _Wiki STM32MP157X-DKX: https://wiki.st.com/stm32mpu/wiki/STM32MP157X-DKX_-_hardware_description
.. _Wiki STM32MP157C-EV1: https://wiki.st.com/stm32mpu/wiki/STM32MP157C-EV1_-_hardware_description
