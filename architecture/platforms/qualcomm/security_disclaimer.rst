.. _qualcomm_security_disclaimer:

####################
Security Disclaimer
####################

	- Using upstream OP-TEE does not make a Qualcomm device secure by
	  default; several platform security settings must still be reviewed
	  and configured correctly for a production build.
	- Some of these settings are configured in Trusted Firmware-A (TF-A)
	  rather than in OP-TEE, such as XPU bus protection on Hoya. Review the
	  platform's TF-A ``BL31`` security configuration alongside this
	  documentation.
	- Integrators remain responsible for validating the complete
	  secure-boot and memory-protection configuration for their product.
