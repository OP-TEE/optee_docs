.. todo::

    Joakim: Feels like this page is a bit malplaced in the structure. Maybe we
    should create a separate "main-section" for this?

.. _platforms_supported:

###################
Platforms supported
###################
Several platforms are supported. In order to manage slight differences between
platforms, a ``PLATFORM_FLAVOR`` flag has been introduced. The ``PLATFORM`` and
``PLATFORM_FLAVOR`` flags define the whole configuration for a chip the where
the Trusted OS runs. Note that there is also a composite form which makes it
possible to append ``PLATFORM_FLAVOR`` directly, by adding a dash in-between the
names. The composite form is shown below for the different boards. For more
specific details about build flags etc, please read
:ref:`configuration_and_flags`. Some platforms have different sub-maintainers,
please refer to the file MAINTAINERS_ for contact details for various platforms.

.. Please keep this list sorted in alphabetic order

.. list-table:: Platforms officially supported in OP-TEE
   :header-rows: 1

   * - Platform
     - Composite PLATFORM flag
     - Publicly available?
     - Maintained?

   * - `AMD Versal Gen 2 <https://www.amd.com/en/products/adaptive-socs-and-fpgas/versal/gen2/prime-series.html>`_
     - ``PLATFORM=versal2``
     - No
     - Yes

   * - `ARM Juno Board <http://www.arm.com/products/tools/development-boards/versatile-express/juno-arm-development-platform.php>`_
     - ``PLATFORM=vexpress-juno``
     - Yes
     - Yes

   * - `Atmel ATSAMA5D2-XULT Board <http://www.atmel.com/tools/atsama5d2-xult.aspx>`_
     - ``PLATFORM=sam``
     - Yes
     - Yes

   * - `Broadcom ns3 <https://www.broadcom.com/products/ethernet-connectivity/smartnic/bcm58800>`_
     - ``PLATFORM=bcm-ns3``
     - No
     - Yes

   * - `DeveloperBox (Socionext Synquacer SC2A11) <https://www.96boards.org/product/developerbox/>`_
     - ``PLATFORM=synquacer``
     - Yes
     - Yes

   * - `FSL ls1021a <http://www.freescale.com/tools/embedded-software-and-tools/hardware-development-tools/tower-development-boards/mcu-and-processor-modules/powerquicc-and-qoriq-modules/qoriq-ls1021a-tower-system-module:TWR-LS1021A?lang_cd=en>`_
     - ``PLATFORM=ls-ls1021atwr``
     - Yes
     - Yes

   * - `NXP ls1043ardb <http://www.nxp.com/products/microcontrollers-and-processors/power-architecture-processors/qoriq-platforms/developer-resources/qoriq-ls1043a-reference-design-board:LS1043A-RDB>`_
     - ``PLATFORM=ls-ls1043ardb``
     - Yes
     - Yes

   * - `NXP ls1046ardb <http://www.nxp.com/products/microcontrollers-and-processors/power-architecture-processors/qoriq-platforms/developer-resources/qoriq-ls1046a-reference-design-board:LS1046A-RDB>`_
     - ``PLATFORM=ls-ls1046ardb``
     - Yes
     - Yes

   * - `NXP ls1012ardb <http://www.nxp.com/products/microcontrollers-and-processors/power-architecture-processors/qoriq-platforms/developer-resources/qoriq-ls1012a-reference-design-board:LS1012A-RDB>`_
     - ``PLATFORM=ls-ls1012ardb``
     - Yes
     - Yes

   * - `NXP ls1028ardb <https://www.nxp.com/pages/design/qoriq-developer-resources/ls1028a-reference-design-board:LS1028ARDB>`_
     - ``PLATFORM=ls-ls1028ardb``
     - Yes
     - Yes

   * - `NXP ls1088ardb <http://www.nxp.com/products/microcontrollers-and-processors/power-architecture-processors/qoriq-platforms/developer-resources/qoriq-ls1088a-reference-design-board:LS1088A-RDB>`_
     - ``PLATFORM=ls-ls1088ardb``
     - Yes
     - Yes

   * - `NXP ls2088ardb <http://www.nxp.com/products/microcontrollers-and-processors/power-architecture-processors/qoriq-platforms/developer-resources/qoriq-ls2088a-reference-design-board:LS2088A-RDB>`_
     - ``PLATFORM=ls-ls2088ardb``
     - Yes
     - Yes

   * - `NXP ls1012afrwy <https://www.nxp.com/support/developer-resources/software-development-tools/qoriq-developer-resources/layerscape-frwy-ls1012a-board:FRWY-LS1012A>`_
     - ``PLATFORM=ls-ls1012afrwy``
     - Yes
     - Yes

   * - `FSL i.MX6 Quad SABRE Lite Board <https://boundarydevices.com/product/sabre-lite-imx6-sbc/>`_
     - ``PLATFORM=imx-mx6qsabrelite``
     - Yes
     - Yes

   * - `FSL i.MX6 Quad SABRE SD Board <http://www.nxp.com/products/software-and-tools/hardware-development-tools/sabre-development-system/sabre-board-for-smart-devices-based-on-the-i.mx-6quad-applications-processors:RD-IMX6Q-SABRE>`_
     - ``PLATFORM=imx-mx6qsabresd``
     - Yes
     - Yes

   * - `SolidRun i.MX6 Quad Hummingboard Edge <https://www.solid-run.com/product/hummingboard-edge-imx6q-wa-h/>`_
     - ``PLATFORM=imx-mx6qhmbedge``
     - Yes
     - Yes

   * - `SolidRun i.MX6 Dual Hummingboard Edge <https://www.solid-run.com/product/hummingboard-edge-imx6d-wa-h/>`_
     - ``PLATFORM=imx-mx6dhmbedge``
     - Yes
     - Yes

   * - `SolidRun i.MX6 Dual Lite Hummingboard Edge <https://www.solid-run.com/product/hummingboard-edge-imx6dl-0c-h/>`_
     - ``PLATFORM=imx-mx6dlhmbedge``
     - Yes
     - Yes

   * - `SolidRun i.MX6 Solo Hummingboard Edge <https://www.solid-run.com/product/hummingboard-edge-imx6s-wa-h/>`_
     - ``PLATFORM=imx-mx6shmbedge``
     - Yes
     - Yes

   * - `FSL i.MX6 UltraLite EVK Board <http://www.freescale.com/products/arm-processors/i.mx-applications-processors-based-on-arm-cores/i.mx-6-processors/i.mx6qp/i.mx6ultralite-evaluation-kit:MCIMX6UL-EVK>`_
     - ``PLATFORM=imx-mx6ulevk``
     - Yes
     - Yes

   * - `NXP i.MX7Dual SabreSD Board <http://www.nxp.com/products/software-and-tools/hardware-development-tools/sabre-development-system/sabre-board-for-smart-devices-based-on-the-i.mx-7dual-applications-processors:MCIMX7SABRE>`_
     - ``PLATFORM=imx-mx7dsabresd``
     - Yes
     - Yes

   * - `NXP i.MX7Solo WaRP7 Board <http://www.nxp.com/products/developer-resources/reference-designs/warp7-next-generation-iot-and-wearable-development-platform:WARP7>`_
     - ``PLATFORM=imx-mx7swarp7``
     - Yes
     - Yes

   * - `NXP i.MX8MQEVK Board <https://www.nxp.com/support/developer-resources/run-time-software/i.mx-developer-resources/evaluation-kit-for-the-i.mx-8m-applications-processor:MCIMX8M-EVK>`_
     - ``PLATFORM=imx-imx8mqevk``
     - Yes
     - Yes

   * - `NXP i.MX8MMEVK Board <https://www.nxp.com/products/processors-and-microcontrollers/arm-based-processors-and-mcus/i.mx-applications-processors/i.mx-8-processors/i.mx-8m-mini-family-arm-cortex-a53-cortex-m4-audio-voice-video:i.MX8MMINI?lang=en&lang_cd=en&>`_
     - ``PLATFORM=imx-imx8mmevk``
     - Yes
     - Yes

   * - `ARM Foundation FVP <https://developer.arm.com/products/system-design/fixed-virtual-platforms>`_
     - ``PLATFORM=vexpress-fvp``
     - Yes
     - Yes

   * - `HiSilicon D02 <http://open-estuary.org/d02-2>`_
     - ``PLATFORM=d02``
     - No
     - Yes

   * - `HiSilicon Hi3519AV100 Demo Board <http://www.hisilicon.com/en/Products/ProductList/Surveillance>`_
     - ``PLATFORM=hisilicon-hi3519av100_demo``
     - No
     - Yes

   * - `HiKey Board (HiSilicon Kirin 620) <https://www.96boards.org/product/hikey>`_
     - ``PLATFORM=hikey` or `PLATFORM=hikey-hikey``
     - Yes
     - Yes

   * - `HiKey960 Board (HiSilicon Kirin 960) <https://www.96boards.org/product/hikey960>`_
     - ``PLATFORM=hikey-hikey960``
     - Yes
     - Yes

   * - `Marvell ARMADA 7K Family <http://www.marvell.com/embedded-processors/armada-70xx/>`_
     - ``PLATFORM=marvell-armada7k8k``
     - Yes
     - Yes

   * - `Marvell ARMADA 8K Family <http://www.marvell.com/embedded-processors/armada-80xx/>`_
     - ``PLATFORM=marvell-armada7k8k``
     - Yes
     - Yes

   * - `Marvell ARMADA 3700 Family <http://www.marvell.com/embedded-processors/armada-3700/>`_
     - ``PLATFORM=marvell-armada3700``
     - Yes
     - Yes

   * - `MediaTek MT8173 EVB Board <https://www.mediatek.com/products/tablets/mt8173>`_
     - ``PLATFORM=mediatek-mt8173``
     - No
     - Yes

   * - `Poplar Board (HiSilicon Hi3798C V200) <https://www.96boards.org/product/poplar>`_
     - ``PLATFORM=poplar``
     - Yes
     - Yes

   * - `QEMU <http://wiki.qemu.org/Main_Page>`_
     - ``PLATFORM=vexpress-qemu_virt``
     - Yes
     - Yes

   * - `QEMUv8 <http://wiki.qemu.org/Main_Page>`_
     - ``PLATFORM=vexpress-qemu_armv8a``
     - Yes
     - Yes

   * - `Raspberry Pi 3 <https://www.raspberrypi.org/products/raspberry-pi-3-model-b>`_
     - ``PLATFORM=rpi3``
     - Yes
     - Yes

   * - `Renesas RCAR <https://www.renesas.com/en-sg/solutions/automotive/products/rcar-h3.html>`_
     - ``PLATFORM=rcar``
     - No
     - Yes

   * - `Renesas RZ/G <https://www.renesas.com/sg/en/products/microcontrollers-microprocessors/rz-cortex-a-mpus/rzg-linux-platform/rzg-marketplace/board-solutions>`_
     - ``PLATFORM=rzg``
     - Yes
     - Yes

   * - `Rockchip PX30 <http://rock-chips.com/a/en/products/rkpower/2018/0709/913.html>`_
     - ``PLATFORM=rockchip-px30``
     - No
     - Yes

   * - `Rockchip RK322X <http://www.rock-chips.com/a/en/products/RK32_Series/2016/1109/799.html>`_
     - ``PLATFORM=rockchip-rk322x``
     - No
     - Yes

   * - `Rockchip RK3399 <http://rock-chips.com/a/en/products/RK33_Series/2016/0419/758.html>`_
     - ``PLATFORM=rockchip-rk3399``
     - Yes
     - Yes

   * - `Rockchip RK3588 <https://www.rock-chips.com/a/en/products/RK35_Series/2022/0926/1660.html>`_
     - ``PLATFORM=rockchip-rk3588``
     - Yes
     - Yes

   * - `STMicroelectronics b2260 - h410 (96boards fmt) <http://www.st.com/web/en/catalog/mmc/FM131/SC999/SS1628/PF258776>`_
     - ``PLATFORM=stm-b2260``
     - No
     - Yes

   * - `STMicroelectronics b2120 - h310 / h410 <http://www.st.com/web/en/catalog/mmc/FM131/SC999/SS1628/PF258776>`_
     - ``PLATFORM=stm-cannes``
     - No
     - Yes

   * - `STMicroelectronics STM32MP157A-DK1 STM32MP157D-DK1 <https://www.st.com/en/evaluation-tools/stm32mp157d-dk1.html>`_
     - ``PLATFORM=stm32mp1-157A_DK1`` or ``PLATFORM=stm32mp1-157A_DK1_SCMI``
     - Yes
     - Yes

   * - `STMicroelectronics STM32MP157C-DK2 STM32MP157F-DK2 <https://www.st.com/en/evaluation-tools/stm32mp157f-dk2.html>`_
     - ``PLATFORM=stm32mp1-157C_DK2`` or ``PLATFORM=stm32mp1-157C_DK2_SCMI``
     - Yes
     - Yes

   * - `STMicroelectronics STM32MP157C-EV1 STM32MP157F-EV1 <https://www.st.com/en/evaluation-tools/stm32mp157f-ev1.html>`_
     - ``PLATFORM=stm32mp1-157C_EV1`` or ``PLATFORM=stm32mp1-157C_EV1_SCMI``
     - Yes
     - Yes

   * - `STMicroelectronics STM32MP135F-DK <https://www.st.com/en/evaluation-tools/stm32mp135f-dk.html>`_
     - ``PLATFORM=stm32mp1-135F_DK``
     - Yes
     - Yes

   * - `Allwinner A64 Pine64 Board <https://www.pine64.org/>`_
     - ``PLATFORM=sunxi-sun50i_a64``
     - Yes
     - Yes

   * - `Texas Instruments AM65x <http://www.ti.com/processors/sitara-arm/am6x-cortex-a53-r5/overview.html>`_
     - ``PLATFORM=k3-am65x``
     - Yes
     - Yes

   * - `Texas Instruments DRA7xx <http://www.ti.com/processors/automotive-processors/drax-infotainment-socs/overview.html>`_
     - ``PLATFORM=ti-dra7xx``
     - Yes
     - Yes

   * - `Texas Instruments AM57xx <http://www.ti.com/processors/sitara/arm-cortex-a15/am57x/overview.html>`_
     - ``PLATFORM=ti-am57xx``
     - Yes
     - Yes

   * - `Texas Instruments AM43xx <http://www.ti.com/processors/sitara/arm-cortex-a9/am438x/overview.html>`_
     - ``PLATFORM=ti-am43xx``
     - Yes
     - Yes

   * - `AMD/Xilinx Versal ACAP <https://www.xilinx.com/products/silicon-devices/acap/versal.html>`_
     - ``PLATFORM=versal``
     - Yes
     - Yes
       
   * - `Xilinx Zynq 7000 ZC702 <http://www.xilinx.com/products/boards-and-kits/ek-z7-zc702-g.html>`_
     - ``PLATFORM=zynq7k-zc702``
     - Yes
     - No (v2.3.0)

   * - `Xilinx Zynq UltraScale+ MPSOC <http://www.xilinx.com/products/silicon-devices/soc/zynq-ultrascale-mpsoc.html>`_
     - ``PLATFORM=zynqmp-zcu102``
     - Yes
     - No (v2.4.0)

   * - `Spreadtrum SC9860 <http://spreadtrum.com/en/SC9860GV.html>`_
     - ``PLATFORM=sprd-sc9860``
     - No
     - No (v2.1.0)

.. _MAINTAINERS: https://github.com/OP-TEE/optee_os/blob/master/MAINTAINERS
