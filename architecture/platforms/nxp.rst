.. _nxp:

###
NXP
###

.. _security_disclaimer:

Security Disclaimer
*******************
	- NXP i.MX processors have various security-relevant modules that may
	  be configured by the customer to effectively secure the device.
	- These security modules vary by the i.MX product family and may include:
		- The **Central Security Unit (CSU)** that manages the system security
		  policy for peripheral access on the SoC.
		- The **Resource Domain Controllers (RDC/XRDC/TRDC)** that provide
		  support for the isolation of peripherals and memory.
		- **Arm® TrustZone®** technology-based memory protection for embedded
		  memories such as the on-chip RAM (OCRAM).
		- The **TrustZone ® Address Space Controller (TZASC)** that protects and
		  secures data in a trusted execution environment.
		- The **AIPSTZ** bridge that provides programmable access protections
		  for both controllers and peripherals.
	- The default security configuration in OP-TEE OS for these security modules
	  is left in an open (non-secure) state because a universal secure
	  configuration that meets all customer requirements is not possible.
	- NXP delivers various open-source software components (NXP OP-TEE OS) for
	  customer enablement, however, these are not provided as secure
	  production-ready implementations.
	- Using OP-TEE OS upstream releases instead of NXP OPTEE-OS releases may
	  have an impact on the features supported and the security level of the
	  i.MX platforms.
	- Customers should optimize the security configuration in OP-TEE OS to lock
	  and secure end products according to their specific security requirements.
	- NXP has documented how to securely configure these security modules in the
	  respective `i.MX SoC Reference and Security manuals <https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors:IMX_HOME>`_
	  and also provides a Security Checklist for the i.MX family to help
	  customers secure end products.
	- For Further assistance please contact your NXP field representative or
	  submit an `NXP Support ticket <https://support.nxp.com/>`_.
