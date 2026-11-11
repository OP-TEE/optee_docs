.. _qualcomm_pil_reset_sequence:

##################################
Qualcomm PIL clock/reset sequences
##################################

Qualcomm remote processors are started through the Peripheral Image Loader
(PIL) flow exposed by the Peripheral Authentication Service (PAS) pseudo TA. The
non-secure world provides the firmware image and memory layout through the OP-TEE
PAS interface. OP-TEE maps the secure controller window, authenticates the image
when the platform requires it, and runs the platform clock/reset sequence needed
to release the remote processor.

The clock/reset operations are intentionally split from the generic PAS command
handling:

	- the PAS core validates the PAS ID, records the firmware location, and calls
	  the platform operation for the selected subsystem;
	- platform PAS files provide subsystem-specific firmware start, shutdown and
	  resource-table operations;
	- the Qualcomm clock driver provides the clock-group hooks used before and
	  after the firmware start operation.

Reset classes
*************

Each subsystem describes which reset class it needs:

	- ``QCOM_PAS_RESET_CLK_FULL`` runs a full clock-driver reset, enables the
	  subsystem clocks, starts the firmware boot FSM, and then performs any
	  post-FSM processor-release step.
	- ``QCOM_PAS_RESET_CLK_ENABLE`` enables the subsystem clocks before starting
	  firmware, but does not run a full reset sequence.
	- ``QCOM_PAS_RESET_NONE`` starts firmware without clock-driver intervention.

The full reset class is used for subsystems that must be forced into a known
hardware state before a boot or shutdown path can proceed.

Full reset sequence
*******************

For a full reset, the PAS core performs this high-level sequence:

	- call ``qcom_clock_pas_reset()`` for the subsystem clock group;
	- call ``qcom_clock_enable()`` to enable the clocks required for register
	  access, boot FSM execution and QDSP6 operation;
	- call the subsystem ``fw_start`` operation to program the firmware entry
	  address and trigger the hardware boot FSM;
	- call ``qcom_clock_enable_pas_processor()`` if the subsystem needs an extra
	  post-FSM step, such as configuring the Q6 PLL and releasing the core.

The implementation keeps timeout-based polling local to each hardware step so
hardware waits do not block forever.

Resource tables
***************

PAS platform files also provide the device-memory resource tables consumed by
the non-secure remoteproc driver. These tables describe the physical ranges
the remote processor may access, including clock, IPC, RPMh, TCSR, SMEM and
subsystem memory windows. The secure side must keep these resources aligned with
the reset and clock sequence: a remote processor can only be released after the
required windows are mapped and the clocks needed for its boot FSM are active.
