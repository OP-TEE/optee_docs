.. _releases:

Releases
########

.. _releases_cadance:

Cadence
*******
New versions of OP-TEE are released four times a year, i.e., quarterly releases.

.. _release_dates:

Release dates
=============
Starting from version 3.10.0 we track the old and also show the releases being
planned for the future in the table below. The dates will tell whether it is an
old, upcoming or future release.

+---------------+--------------+
| Version       | Release date |
+===============+==============+
| OP-TEE 4.12.0 | 16/Oct/26    |
+---------------+--------------+
| OP-TEE 4.11.0 | 17/Jul/26    |
+---------------+--------------+
| OP-TEE 4.10.0 | 17/Apr/26    |
+---------------+--------------+
| OP-TEE 4.9.0  | 16/Jan/26    |
+---------------+--------------+
| OP-TEE 4.8.0  | 17/Oct/25    |
+---------------+--------------+
| OP-TEE 4.7.0  | 11/Jul/25    |
+---------------+--------------+
| OP-TEE 4.6.0  | 11/Apr/25    |
+---------------+--------------+
| OP-TEE 4.5.0  | 17/Jan/25    |
+---------------+--------------+
| OP-TEE 4.4.0  | 18/Oct/24    |
+---------------+--------------+
| OP-TEE 4.3.0  | 12/Jul/24    |
+---------------+--------------+
| OP-TEE 4.2.0  | 12/Apr/24    |
+---------------+--------------+
| OP-TEE 4.1.0  | 19/Jan/24    |
+---------------+--------------+
| OP-TEE 4.0.0  | 20/Oct/23    |
+---------------+--------------+
| OP-TEE 3.22.0 | 14/Jul/23    |
+---------------+--------------+
| OP-TEE 3.21.0 | 14/Apr/23    |
+---------------+--------------+
| OP-TEE 3.20.0 | 20/Jan/23    |
+---------------+--------------+
| OP-TEE 3.19.0 | 14/Oct/22    |
+---------------+--------------+
| OP-TEE 3.18.0 | 15/July/22   |
+---------------+--------------+
| OP-TEE 3.17.0 | 15/Apr/22    |
+---------------+--------------+
| OP-TEE 3.16.0 | 28/Jan/22    |
+---------------+--------------+
| OP-TEE 3.15.0 | 18/Oct/21    |
+---------------+--------------+
| OP-TEE 3.14.0 | 16/July/21   |
+---------------+--------------+
| OP-TEE 3.13.0 | 30/Apr/21    |
+---------------+--------------+
| OP-TEE 3.12.0 | 20/Jan/21    |
+---------------+--------------+
| OP-TEE 3.11.0 | 16/Oct/20    |
+---------------+--------------+
| OP-TEE 3.10.0 | 21/Aug/20    |
+---------------+--------------+

.. _releases_changelog:

Changelog
*********
The changelog is stored in the :ref:`optee_os` git (CHANGELOG.md_). There you
can see what has been done between the different releases in terms of commits as
well as pull requests.

.. _releases_versioning_schema:

Versioning schema
*****************
OP-TEE follows `Semantic Versioning 2.0.0`_. What that means in practice is well
described at the link just shown.

.. _releases_release_procedure:

Release procedure
*****************
There are certain steps that needs to be done when making a release. This
checklist here serves as guidance to the one in charge of making a new release.
Start with this 3 weeks before the targeted release date.

tl;dr
=====
.. list-table:: Short version of the OP-TEE release procedure
    :widths: 60 300 10
    :header-rows: 1

    * - When
        (Tminus)
      - Action
      - Example

    * - 3w
      - Create release pull request
      - `PR#3099`_

    * - 3w
      - Inform maintainers about upcoming release
      -

    * - 2w
      - Increment the revision number in `mk/config.mk`_
      - ``CFG_OPTEE_REVISION_MAJOR ?= 3`` ``CFG_OPTEE_REVISION_MINOR ?= x``

    * - 2w
      - Create release candidate tag in optee_* + build.git
      - git tag -a 3.x.y-rc1 -m "3.x.y-rc1"

    * - 2w
      - Let maintainers know about the release candidate tag
      -

    * - 2w
      - Test platform builds / devices
      -

    * - Release day
      - Update CHANGELOG.md_
      - `changelog example`_

    * - Release day
      - Collect/merge ``Tested-By`` tags
      - `commit example`_

    * - Release day
      - Create release tag in optee_* + build.git + linux.git
      - | git tag -a 3.x.y -m "3.x.y"
        | git tag -a optee-3.x.y -m "optee-3.x.y" # (Linux)

    * - Release day
      - Create release branch in :ref:`manifest`
      - git checkout -b 3.x.y origin/master

    * - Release day
      - Update manifest XML-files
      - `3.6.0 stable`_

    * - Release day
      - Inform maintainers and stakeholder that release has been completed.
      -


Long version
============

    1. Create a "release pull request" at GitHub ought to collect ``Tested-By``
       tags from various maintainers. As an example, see `PR#3099`_.

    2. Send email to all maintainers to let them know about the upcoming
       release. The addresses to the maintainers can be found in the
       MAINTAINERS_ file.

       .. hint::
            With this command you will get all email addresses

            .. code-block:: bash

                $ scripts/get_maintainer.py --release-to

    3. Increment the revision number in `mk/config.mk`:
       ``CFG_OPTEE_REVISION_MAJOR`` and ``CFG_OPTEE_REVISION_MINOR``. These
       values are made available to TAs and to the Normal World driver at boot
       time.

    4. Create a release candidate (RC) tag (annotated tag, i.e., ``git tag -a
       3.x.y-rc1 -m "3.x.y-rc1"``) in the following gits
       ``optee_*`` and ``build.git``. One way to do it is like this

       .. code-block:: bash

            $ export VER=3.x.y-rc1
            $ for d in optee* build; do ( cd $d; git tag -a $VER -m $VER ); done
            $ for d in optee* build; do ( cd $d; git push origin $VER ); done


    5. Send a follow up email to all maintainers to let them know that there is
       a release tag ready to be tested on their devices for the platforms that
       they are maintaining.

    6. In case major regressions are found, then fix those and create a another
       release candidate tag (i.e., repeat step 3 and 4 until there are no
       remaining issues left).

    7. On release day: Update CHANGELOG.md_ see this `changelog example`_ to see
       how that should look like.

    8. Collect all tags (``Tested-By`` etc) from maintainers and use those in
       the commit message, for an example see this `commit example`_.

    9. Create a release tag (annotated tag, i.e., ``git tag -a 3.x.y -m
       "3.x.y"``) in the following gits ``optee_*`` and ``build.git``. Tag the
       tip of the ``optee`` branch in ``linux.git``, the name of the tag has
       to be prefixed with ``optee-`` to avoid confusions. For instance:
       ``git tag -a optee-3.x.y -m "optee-3.x.y"``.

       .. hint::

            You can use the same steps as in step 4, when creating the tags.

    10. Create a new branch in :ref:`manifest` from ``master`` where the name
        corresponds to the release you are preparing. I.e., ``git checkout -b
        3.x.y origin/master``.


    11. Update all :ref:`manifest` XML-files in the :ref:`manifest` git, so they
        refer to the tag in the release we are working with (see `3.6.0 stable`_
        commit as an example). This can be done with the make_stable.sh_ script.
        Now it is also time to push the new branch and tag it. Example:

       .. code-block:: bash

            $ export VER=3.x.y
            $ cd manifest
            $ ./make_stable.sh -o -r $VER
            $ git diff  # make sure everything looks good
            $ git commit -a -m "OP-TEE $VER stable"
            $ git remote add upstream git@github.com:OP-TEE/manifest
            $ git push upstream
            $ git tag -a $VER -m $VER
            $ git push upstream tag $VER

    12. Send a last email to maintainers and other stakeholders telling that the
        release has been completed.


.. _3.6.0 stable: https://github.com/OP-TEE/manifest/commit/f181e959c21baddce82552104daf81a25f8fd898
.. _CHANGELOG.md: https://github.com/OP-TEE/optee_os/blob/master/CHANGELOG.md
.. _changelog example: https://github.com/OP-TEE/optee_os/commit/f398d4923da875370149ffee45c963d7adb41495#diff-4ac32a78649ca5bdd8e0ba38b7006a1e
.. _commit example: https://github.com/OP-TEE/optee_os/commit/f398d4923da875370149ffee45c963d7adb41495
.. _MAINTAINERS: https://github.com/OP-TEE/optee_os/blob/master/MAINTAINERS
.. _make_stable.sh: https://github.com/OP-TEE/manifest/blob/master/make_stable.sh
.. _PR#3099: https://github.com/OP-TEE/optee_os/pull/3099
.. _Semantic Versioning 2.0.0: https://semver.org
.. _mk/config.mk: https://github.com/OP-TEE/optee_os/blob/master/mk/config.mk
