.. _releases:

Releases
########

.. _releases_cadance:

Cadence
*******
New versions of OP-TEE are released four times a year, i.e., quarterly releases.
The releases have historically lined up with Linaro Connect events. I.e., a
release has been made right after Linaro Connect and one release somewhere
between two Linaro Connects. I.e., typically there has been releases in January,
April, July and October.

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
Roughly start with this 2-3 weeks before the targeted release date.

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

    * - 1w
      - Increment the revision number in `mk/config.mk`_
      - ``CFG_OPTEE_REVISION_MAJOR ?= 3`` ``CFG_OPTEE_REVISION_MINOR ?= x``

    * - 1w
      - Create release candidate tag in optee_* + build.git
      - git tag -a 3.x.y-rc1 -m "3.x.y-rc1"

    * - 1w
      - Let maintainers know about the release candidate tag
      -

    * - 1w
      - Test platform builds / devices
      -

    * - Release day
      - Update CHANGELOG.md_
      - `changelog example`_

    * - Release day
      - Collect/merge ``Tested-By`` tags
      - `commit example`_

    * - Release day
      - Create release tag in optee_* + build.git
      - git tag -a 3.x.y -m "3.x.y"

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
            With this command in bash you will get all email addresses

            .. code-block:: bash

                $ cat MAINTAINERS | grep '[RM]:.*<.*@.*>' | \
                  sed 's/>.*/>/' | sed 's/.:\t//' | sort | uniq

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
       "3.x.y"``) in the following gits ``optee_*`` and ``build.git``.

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
            $ git tag -s -a $VER -m $VER
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
