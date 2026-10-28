.. _qualcomm_wildcat:

####################
Wildcat architecture
####################

The Wildcat family is built around Qualcomm's Oryon CPU. It currently covers
the ``nord`` chipset.

On top of the :ref:`common platform features <qualcomm>`, Wildcat enables a
GICv4 interrupt controller (``CFG_ARM_GICV4``). Support is otherwise minimal
at this stage; additional drivers and services will be documented as they
are enabled.

Chipsets
********

nord
====
18-core configuration.
