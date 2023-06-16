.. _contact:

#######
Contact
#######

GitHub
******
Our preference is to use GitHub for communication. The reason for that is that
it is an open source project, so there should be no real reason to hide
discussions from other people. GitHub also makes it possible for anyone to chime
in into discussion etc. So besides sending patches as pull requests on GitHub we
also encourage people to use the "issues_" to report bugs, give suggestions, ask
questions etc.

Please try to use the "issues" in the relevant git. I.e., if you want to discuss
something related to optee_client, then use "issues" at :ref:`optee_client` and
so on. If you have a general question etc about OP-TEE that doesn't really
belong to a specific git, then please use `issues at optee_os`_ in that case.

Email
*****
You can reach the :ref:`core_team` by sending an email to
``op-tee[at]lists.trustedfirmware.org``. However note that it's a public
mailinglist and **not** just TrustedFirmware engineers behind that email
address.

For pure Linux kernel patches, please use the appropriate Linux kernel
mailinglist, basically run the ``get_maintainer.pl`` script in the Linux kernel
tree to figure out where to send your patches.

.. code-block:: bash

    $ cd <linux-kernel>
    $ ./scripts/get_maintainer.pl drivers/tee/


IRC
***
Some of the OP-TEE developers can be reached at Freenode (``chat.freenode.net``)
at channel ``#linaro-security``. Having that said, the activity there is a bit
limited, so it is probably **not** the best place to discuss OP-TEE.

.. _vulnerability_reporting:

Vulnerability reporting
***********************
As part of the TrustedFirmware.org organization, the OP-TEE project uses the
security incident procedure outlined on the `TrustedFirmware.org security
incident`_ page. We offer two methods for reporting security issues. The first
is the traditional method of sending email to the addresses listed on the
`Mailing Aliases`_ page. The alternative method is through GitHub's `GitHub
Security Advisories`_ page for OP-TEE.

We prefer the `GitHub Security Advisories`_ page because they simplify the
sharing and communication of reports. However, this also requires a GitHub
account and we recognize that not everyone can or has the ability to report
security issues via GitHub; therefore, we also accept reports via email.

Note that OP-TEE is a reference implementation for developers and device
manufacturers and by being a reference implementation it is not always running a
secure device configuration by default (see :ref:`platform_ports` for more
information). Therefore we ask people to think twice whether the security
incident report should go to:

 a) The OP-TEE project? Is it an issue in the generic code?
 b) The chipmaker? Does it only affect a certain platform? Is it a configuration described only under NDA?
 c) The ones making the end product? Is the issue only present on a certain device?

In some cases, the OP-TEE team works directly with chipmakers. However, it is
not uncommon for products to be manufactured using OP-TEE without the OP-TEE
project's knowledge. In such instances, we recommend sending the security issue
report to the manufacturer of the final product, who should then, if necessary,
contact the OP-TEE project and/or the chipmaker.

.. _core_team:

Core Team
*********
The core team consists of TrustedFirmware.org engineers. See also "THE REST" in
the `OP-TEE MAINTAINERS`_ file, which oversees the essential activities, such as
performing releases, merging patches, and being the first to respond to security
incidents.

.. _GitHub Security Advisories: https://github.com/OP-TEE/optee_os/security/advisories
.. _issues: https://help.github.com/articles/about-issues/
.. _issues at optee_os: https://github.com/OP-TEE/optee_os/issues
.. _Mailing Aliases: https://developer.trustedfirmware.org/w/collaboration/security_center/mailing_aliases
.. _OP-TEE MAINTAINERS: https://github.com/OP-TEE/optee_os/blob/master/MAINTAINERS
.. _TrustedFirmware.org security incident: https://developer.trustedfirmware.org/w/collaboration/security_center
