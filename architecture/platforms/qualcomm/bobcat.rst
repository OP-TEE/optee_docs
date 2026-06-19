.. _qualcomm_bobcat:

###################
Bobcat architecture
###################

The Bobcat family targets Qualcomm networking processors. It currently covers
the ``ipq96xx`` and ``ipq52xx`` chipsets.

On top of the :ref:`common platform support <qualcomm>`, Bobcat currently
provides a minimal, family-specific configuration. Additional drivers and
services will be enabled as support evolves.

Chipsets
********

ipq96xx
=======
5-core configuration with a GICv3 interrupt controller (``CFG_ARM_GICV3``).

ipq52xx
=======
4-core configuration.
