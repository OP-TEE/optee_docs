.. _disclosure_policy:

#################
Disclosure policy
#################
When a vulnerability has been reported (see :ref:`vulnerability_reporting`) to
the :ref:`core_team`, it is up to them to implement mitigations and fixes as
well as report back to stakeholders in a responsible way. This page describes
the responsible disclosure policy that applies to the OP-TEE project.

.. note::
    The "`core team`_" in Linaro (who owns the OP-TEE project) consists of
    engineers directly employed by Linaro as well as engineers employed by
    companies who are members of Linaro.

Rules
*****
To have some kind of ground to stand on we have defined a set of rules and
conditions that applies both when it comes to being a taker of information as
well as being reporter of security issues. It should be noted that it is hard to
write rules that you can follow to 100%, since depending on the type of security
issues being dealt with it might or might not be possible for the core team and
Linaro to re-distribute the information right away.

As an example of when we couldn't follow our rules and disclosure policy was
when we got informed (under NDA) about the Spectre and Meltdown issues (this was
before it was public knowledge). That was considered so sensitive that we
weren't even allowed to share or discuss this outside Linaro (employees). But in
general, we strive and try to do our best to follow the rules etc that have been
defined on this particular page.

Receiving information
=====================
The one receiving information about and fixes related to OP-TEE security
vulnerabilities must follow these rules:

    1. The **receiver** of vulnerability information and/or security fixes
       shared by the core team and Linaro are **not allowed** to share,
       re-distribute or otherwise spread knowledge about the issues and security
       fixes outside their own company until the disclosure deadline has passed
       and the information is publicly available.

       .. note::

        If the receiver still insists to share it with other people/companies he
        must first get approval from the core team and Linaro to do so.

.. _reporting_issues:

Reporting issues
================
The one reporting security vulnerabilities to the core team and Linaro are asked
to do it under the conditions mentioned below. It might seem like a long list,
but we hope that it won't scare people away from reporting issues. It's mostly
common sense and also aims to rule out questions that otherwise might come to
mind. In short, the rules by default gives the core team and Linaro the power to
decide what to do with the reported issue if nothing else has been agreed
between them and the reporter.

    1. If nothing else has been agreed between the reporter and the core team
       and Linaro, then the rules and information as stated on this page
       applies.

        1.1. This means that the core team and Linaro will re-distribute the
        information to the stakeholders according to the plan described further
        down here.

        1.2. This also means that patches etc will be submitted to the upstream
        project based on the proposed disclosure day that will be given to the
        reporter after initial investigation.

    2. By default, the information about the reported issue(s) will be shared
       within the core team (see the note about the core team at the beginning
       of the page). If you as a reporter **aren't** OK with that, then you must
       inform us about that when reporting the issue.

    3. By default, the core team and Linaro decides whether there should be a
       CVE created or not. If the reporter insist on having a CVE created, then
       this should be expressed when doing the reporting.

    4. The core team and Linaro have the rights to involve other experts to help
       us with mitigations and patches. If you as a reporter **aren't** OK with
       that, then you must inform us about that when reporting the issue.

    5. Reporting security issues under NDA should be seen as a last resort
       thing. If/when that happens, then we will come up with a mutual agreement
       on a disclosure plan.

    6. It is appreciated if the reporter have estimated some initial severity
       scoring as described further down on this page. This is mainly to get an
       indication whether we share the same view about the severity or not.


Trusted Stakeholders
********************
The core team keeps track of companies and maintainers who are considered as
trustworthy OP-TEE users. This is a vetted list and people from companies can
only be added to that list after first talking to the core team. In short what
is required to be added to that list is:

    - A justification of why you need to know about security issues and should
      have access to security fixes before they are going public.

    - A company email address (we do not accept gmail, yahoo and similar
      addresses).

    - You accept our disclosure policy rules (as described at here).

.. note::
    The core team and Linaro have the rights to deny anyone to be on this list.
    We also have the rights to remove people on the list if there should be a
    reason to do so.

Disclosure deadline
*******************
By default we are following the industry standard with 90-days disclosure
deadline. This applies both when **we** find security issues that needs to be
fixed in the upstream project, as well as when we are the ones reporting issues
found in vendor trees (forks of OP-TEE). The reason for 90-days is to give
companies enough time to patch and deploy updated software to their devices.

Likewise we are going to propose a 90-days disclosure deadline for issues that
are being reported to us, that we are supposed to fix.

However, for issues that falls in the severity category 'low' and in some cases
'medium' (see :ref:`severity_table` below), we have the rights to decide whether
to upstream patches as soon as they are ready. If the reporter or the some of
the trustworthy stakeholders knowing about the security issue disagrees, then
they must inform the core team and Linaro about it as soon as possible and then
we will come up with an alternate plan.

0day exploits
=============
This is a previously unknown and unpatched vulnerability which is been used
actively in the wild. As a consequence of that we believe that 0day_ exploits
require a much more urgent action. I.e., a fix or some kind of mitigation that
limits the damage needs to be created as soon as possible. Our target for such
fixes and mitigations are within 14 days from the day when we learned about the
0day exploit (full weeks, including weekends).

Issue process
*************
For **regular** security issues (non 0day) we follow the flow chart below. Note
that the orange path is when it is a **low** (and maybe medium) severity issue
we are dealing with, so that is a special case with an alternate path.

.. graphviz::

    digraph issue_process {
        start [label="Issue reported\nDay 1\n90 day counter starts", shape="box", style=rounded];
        end [label="Day 90", shape="box", style=rounded];
        create [label="Create mitigations"];
        inform [label="Inform stakeholders"];
        patch_ready [label="Patch ready"];
        go_public [label="Update security advisories"];
        upstream_fixes [label="Upstream Fixes"];
        medhigh_prio [label="Severity >= Low/Medium?", shape="parallelogram"];
        create_cve [label="Create CVE"];
        update_cve [label="Update CVE\n(if created)"];


        start -> create;
        start -> inform;

        create -> medhigh_prio;
        medhigh_prio -> create_cve [label="Yes"];
        medhigh_prio -> upstream_fixes [label="No", color="orange"];

        create -> patch_ready;
        patch_ready -> inform [label="Share fixes"];
        patch_ready -> end;
        patch_ready -> medhigh_prio [label="Check if patch should go upstream directly", color="orange"];

        end -> inform;
        end -> go_public;
        end -> upstream_fixes;
        end -> update_cve;
    }

For **0day** exploits we follow this flow chart:

.. graphviz::

    digraph issue_process {
        start [label="\0day issue reported\nDay 1\n14 day counter starts", shape="box", style=rounded];
        end [label="Day 14", shape="box", style=rounded];
        create [label="Create mitigations"];
        inform [label="Inform stakeholders"];
        patch_ready [label="Patch ready"];
        go_public [label="Update security advisories"];
        upstream_fixes [label="Upstream Fixes"];
        medhigh_prio [label="Severity >= Medium?", shape="parallelogram"];
        create_cve [label="Create CVE"];
        update_cve [label="Update CVE"];

        start -> create;
        start -> inform;

        create -> medhigh_prio;
        medhigh_prio -> create_cve [label="Yes"];

        create -> patch_ready;
        patch_ready -> inform [label="Share fixes"];
        patch_ready -> end;

        end -> inform;
        end -> go_public;
        end -> upstream_fixes;
        end -> update_cve;
    }


Recognition
***********
Once the disclosure deadline has passed and information and mitigations will go
public we want to give credits to the ones finding, reporting and fixing the
issues. Typically that is given in two ways. One is in textual form at our
`security advisories`_ page and the other way is directly in patches applied on
the upstream project in questions.

For patches we prefer having a real physical person being mentioned (see
*Reported-by* and *Suggested-by* in the example below), but also a company name
or group could be used if it was a joint effort finding the security issue or if
the person finding the issue prefer not being mentioned directly for some
reason. A patch would typically look like this:

.. code-block:: none
    :emphasize-lines: 11,12

    core: fixes privilege escalation

    By doing X, one was able to exploit a privilege escalation
    vulnerability. By changing Y this is no longer a security
    issue.

    Fixes CVE-20xx-YYYY

    Signed-off-by: John Doe <john.doe@foobar.org>
    Reviewed-by: Richard Roe <richard.roe@foobar.org>
    Reported-by: Jane Doe <jane.doe@notable-hackers.com>
    Suggested-by: Jane Doe <jane.doe@notable-hackers.com>

CVE
***
If there is a need to request a CVE identifier, then the `Distributed Weakness
Filing Project`_ should be used. At that page you will find the current link to
the DWF project.

Severity scoring
****************
When deciding the severity for a vulnerability we start out by doing a scoring
similar to the DREAD_ scoring system, but tweaked for OP-TEE purposes. This
mainly serves as a guide to get some kind of indication of the severity. The
final severity is decided on case by case basis.

.. note::
    A DREAD score can change over time. The initial analysis could give a
    certain score, but later on when a vulnerability is well known and exploits
    are readily available the score will be different (ususally more severe).

**Damage Potential**

This should give an answer to much damage is caused if the vulnerability is
exploited.

.. list-table::
    :widths: 1 20
    :header-rows: 1

    * - Score
      - Damange potential

    * - 0
      - No damage.

    * - 1
      - Normal World User space is compromised and could leak sensitive data.

    * - 1
      - Denial of service from Normal World.

    * - 2
      - Normal World Linux kernel space is compromised and could leak sensitive
        data.

    * - 5
      - TEE Trusted Application compromised and could leak data only accessible
        by the Trusted Application.

    * - 7
      - TEE core (kernel space) compromised and leaking trivial information.

    * - 9
      - TEE core (kernel space) compromised and leaking sensitive information.

    * - 10
      - TEE fully compromised and the attacker in full control.

**Reproducibility**

This describes how easy (or hard) it is to reproduce the attack.

.. list-table::
    :widths: 1 20
    :header-rows: 1

    * - Score
      - Reproducibility

    * - 0
      - Not reproducible.

    * - 1
      - No proven attack exists.

    * - 1
      - The attack is very difficult to reproduce, even with knowledge of the
        security hole (requires special lab equipment for example)

    * - 2
      - Proof of concept attack exists, but only works in a specially crafted,
        non-standard configuration.

    * - 4
      - The attack can be reproduced, but only with tooling / software /
        knowledge that has **not** been made public (typically the one finding
        the security issue have created a tool, which hasn't been released yet).

    * - 9
      - The attack can be reproduced, but only with tooling (JTAG,
        ChipWhisperer_ etc) / software / knowledge that is readily available to
        anyone.

    * - 10
      - The attack can be reproduced every time by a novice user without any
        need for extra tools.

**Exploitability**

This should answer how easy it is to launch an attack.

.. list-table::
    :widths: 1 20
    :header-rows: 1

    * - Score
      - Exploitability

    * - 0
      - Not exploitable.

    * - 1
      - Theoretically exploitable (even with knowledge, there seems to be no
        viable path for a real exploit).

    * - 7
      - Only authenticated user(s) can make the attack.

    * - 8
      - A skilled programmer with in-depth knowledge could make the attack.

    * - 9
      - A novice programmer could make the attack in a short time.

    * - 10
      - A novice user could make the attack in a short time (exploits readily
        available on internet and/or integrated in known hacker/pen-testing
        tools).

**Affected Users**

This should give a rough answer to how many people are affected by a successful
attack.

.. list-table::
    :widths: 1 20
    :header-rows: 1

    * - Score
      - Affected Users

    * - 0
      - No users affected.

    * - 1
      - All users, running a debug/developer configuration.

    * - 1
      - A single user.

    * - 10
      - All users, running a release configuration (key customers).

**Discoverability**

This should answer how easy it is to discover the threat.

.. list-table::
    :widths: 1 20
    :header-rows: 1

    * - Score
      - Discoverability

    * - 0
      - Not discoverable.

    * - 1
      - The vulnerability would require other successful exploits in order to be
        able to discover this bug.

    * - 2
      - The bug is obscure, and it is unlikely that users will work out damage
        potential.

    * - 5
      - Information explaining the attack exists, but is only shared with a
        small group of people (and it is not intended to be shared publicly in a
        foreseeable time or until mitigations has been merged).

    * - 10
      - Published information explains the attack.

.. _severity_table:

Severity table
==============
Based on the DREAD score, we get some kind of indication of the severity. In the
table below you can see how we are mapping things between a DREAD score and
severity.

.. list-table::
    :widths: 1 4 1 20
    :header-rows: 1

    * - Severity
      - Score
      - CVE?
      - Comment

    * - No risk
      - [0, 1)
      - No CVE created.
      - This is not considered as a security issue, it's a regular bug.

    * - Low
      - [1, 4)
      - No CVE created.
      - This could be seen as a security issue, but could probably be treated as
        general bug.

    * - Medium
      - [4, 7)
      - Depends.
      - This is a security issue, but on the lower side of the score it might be
        treated as a bug. For the higher end it is likely that a CVE will be
        created.

    * - High
      - [7, 9)
      - CVE created.
      - It is definitely a security issue.

    * - Critical
      - [9, 10]
      - CVE created.
      - It is definitely a security issue, very urgent to start working with
        mitigations etc.


Example
=======
To have a better understanding how this would look like in practice, let's show
a couple of examples.

**Example 1** - Spectre v2 - Branch Target Injection (CVE-2017-5715_)

Note that this example should be seed from a TrustZone / TEE point of view.

    - **D**: What damage could it cause?
        - TEE leaking sensitive data, i.e., 9.

    - **R**: Easy to reproduce?
        - No proven attack exists on TrustZone/TEE software, i.e, 1.

    - **E**: Easy to launch the attack?
        - Theoretically exploitable, i.e., 1

    - **A**: How many users would be affected by a successful attack?
        - All users, i.e., 10.

    - **D**: How easy is it to discover this issue?
        - It's public information, i.e., 10.

This gives the score: (9 + 1 + 1 + 10 + 10) / 5 = **6.2** which *indicates* that
this would a bit on the higher end of medium severity.

**Example 2** - Bellcore attack on OP-TEE (CVE-2017-1000412_)

    - **D**: What damage could it cause?
        - TEE leaking sensitive data (private key used to sign and verify
          Trusted Applications), i.e., 9.

    - **R**: Easy to reproduce?
        - With a ChipWhisperer_ (readily available) it would be possible for a
          somewhat skilled engineer to do this on their own on a device running
          OP-TEE, i.e., 9.

    - **E**: Easy to launch the attack?
        - A skilled engineer with in-depth knowledge could make the attack, i.e., 8.

    - **A**: How many users would be affected by a successful attack?
        - All users, i.e., 10.

    - **D**: How easy is it to discover this issue?
        - It's public information, i.e., 10.

This gives the score: (9 + 9 + 8 + 10 + 10) / 5 = **9.2** which *indicates* that
this would be a critical issue.


.. _0day: https://en.wikipedia.org/wiki/Zero-day_(computing)
.. _ChipWhisperer: https://newae.com/tools/chipwhisperer/
.. _core team: https://github.com/orgs/OP-TEE/teams/linaro/members
.. _Distributed Weakness Filing Project: https://cve.mitre.org/cve/request_id.html
.. _DREAD: https://wiki.openstack.org/wiki/Security/OSSA-Metrics#DREAD
.. _CVE-2017-5715: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5715
.. _CVE-2017-1000412: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-1000412
.. _security advisories: https://www.op-tee.org/security-advisories/
