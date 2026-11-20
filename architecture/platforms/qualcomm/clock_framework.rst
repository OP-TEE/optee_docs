.. _qualcomm_clock_framework:

#########################
Qualcomm clock framework
#########################

Qualcomm platforms use the common OP-TEE clock framework
(``CFG_DRIVERS_CLK``) with a Qualcomm-specific driver layer
(``CFG_DRIVERS_QCOM_CLK``). The driver owns secure-world access to clock
controller registers that must not be programmed by the non-secure world during
secure service or remote-processor bring-up flows.

The framework is intentionally data-driven. Common code implements the clock
operations, while each chipset provides the register addresses, parent mappings
and rate tables that describe its clock topology.

Clock model
***********

The Qualcomm driver models the hardware as separate OP-TEE ``struct clk``
objects:

	- **PLL vote clocks** represent shared PLL sources. Enabling the clock sets
	  the appropriate vote bit; disabling it clears the vote when the common clock
	  framework reference count reaches zero.
	- **RCG clocks** represent root clock generators. They program the parent
	  source, divider and optional M/N/D values through the CMD_RCGR register bank.
	- **Branch clocks** represent CBCR gates. A branch can either set the CBCR
	  enable bit directly or use a shared branch-vote register when the hardware
	  exposes one.

Combined consumer clocks, such as QUPv3 serial-engine clocks, are represented
as a branch clock parented by its RCG. This lets the common clock framework keep
standard enable reference counts while the Qualcomm driver performs the
hardware-specific parent, rate and rail-vote sequencing.

Chipset data
************

A chipset supplies a ``struct clk_cfg`` table with:

	- the MMIO windows that contain the clock registers;
	- the shared PLL vote clocks available to the chipset;
	- the RCG descriptors, including rate plans sorted by output frequency;
	- the RCG parent-map values used by each hardware SRC_SEL field;
	- the branch clock descriptors and their optional vote registers.

Because Qualcomm platforms do not use a secure device tree for clocks, the
driver registers clocks by name. Secure-world consumers use
``qcom_clk_get_by_name()`` to acquire a clock and then use the common
``clk_*()`` APIs for rate selection and enable/disable.

Rate changes
************

For a requested rate, the RCG code selects the first rate-table entry greater
than or equal to the request. When the clock is disabled, the driver records the
selected parent and rate, and the hardware programming is deferred until the
clock is enabled.

When an enabled RCG changes rate, the driver performs the sequence in a safe
order:

	- raise the required CX/MX voltage vote before selecting a faster rate;
	- vote for the new parent PLL before switching the RCG source;
	- program the CFG/M/N/D registers and wait for the CMD update bit to clear;
	- release the old parent PLL vote after the switch is live;
	- lower the voltage vote only after the rate change has completed.

On disable, an RCG is parked on XO before the driver drops its voltage vote.

RPMh rail votes
***************

Some clock rates require a minimum CX or MX voltage corner. The Qualcomm clock
driver keeps per-corner demand counts and sends RPMh active-set votes through
the secure RPMh client. Corner-to-RPMh-level mappings are read from Command DB:
the resource address comes from the resource entry and the level table comes
from the auxiliary data blob.

CX is mandatory for this vote path. MX is optional because not every chipset has
an MX rail exposed in Command DB. If present, CX and MX are resolved against
their own level tables and voted independently.

DFS support
***********

Dynamic frequency scaling (DFS) RCGs expose performance-state banks. The driver
programs each rate-table entry that has a DFS index into the matching DFS bank
and then enables hardware DFS ownership. After DFS is enabled, clients are not
expected to call ``clk_set_rate()`` for that RCG; consumers can query supported
DFS rates and indexes with ``qcom_clk_get_dfs_rates_array()``.

PAS clocks
**********

The PAS pseudo TA uses the Qualcomm clock API through clock groups such as
``QCOM_CLKS_TURING``, ``QCOM_CLKS_LPASS`` and ``QCOM_CLKS_GPDSP0``. These group
hooks are implemented per chipset because the reset, bus-clock and QDSP6 PLL
requirements differ by subsystem. See :ref:`qualcomm_pil_reset_sequence` for
the reset and processor-release sequencing used around PAS firmware start.
