.. _security_scope:

Security scope
##############

This document defines the security scope of OP-TEE. It clarifies the trust
model, the types of systems considered in scope, and the precise criteria
used to determine whether a bug qualifies as a security vulnerability.

Trust model
***********

OP-TEE executes at secure kernel mode (typically S-EL1) and trusts the
secure higher privilege or exception levels (typically S-EL2 and EL3). The
secure interfaces of the hardware are trusted; for example, devices
accessible only by the secure world, or devices with a secure interface
like the GIC.

The memory assigned to OP-TEE is trusted.

It is understood that hardware can be attacked with glitching or
Microarchitectural Side-Channels.

Target system
*************

Issues must target a real end-user system with a production configuration.
Open or modified development hardware is out of scope.

What is a Security Issue?
*************************

A bug is a security issue if it allows an actor to violate the
confidentiality, integrity, or availability of any asset outside of its
authorized isolation domain. This includes lateral isolation failures and
confused deputy attacks: OP-TEE Core must not be induced to act on behalf
of one actor (Normal World or one TA) in a way that affects an unrelated
peer.

The `TEE Protection Profile specification`_, Figure 2-1 TEE Software
Architecture, has a useful overview of the involved components and
isolation boundaries.

Availability violations are security issues only if they impact resources
beyond the attacker’s own domain.

Side-channel issues exist when well-known mitigations are missing (e.g.
non-constant-time crypto) and secrets can be inferred across isolation
boundaries.

Requisites
**********

Most bugs have a prerequisite to be triggered. This might be a certain
state in the system, a specific configuration of the system, and similar
factors. If the attacker can provide the prerequisites to trigger the bug,
we most likely have a security issue.

A malicious TA, as a prerequisite, is disqualified because TA signing keys
define the trust boundary. A party with signing authority is already within
the trusted computing base.

Proving the security issue
**************************

A piece of code that demonstrates how the security issue can be triggered
is a compelling argument and helps the security team triage the issue. To
be effective, such demonstration code must be clean and easy to read.

Given the high complexity of the environments and hardware requirements
involved, the OP-TEE security team is not expected to compile and run the
provided code. Instead, the code should serve as a clear, logical
illustration of the flaw within the OP-TEE source code, allowing the team
to verify the vulnerability through analysis.

Not all bugs are easily demonstrated; for instance, those based on
glitching or Microarchitectural Side-Channels, mentioned above.

.. _TEE Protection Profile specification: https://globalplatform.org/specs-library/tee-protection-profile-v1-3/
