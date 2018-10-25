.. _contribute:

##########
Contribute
##########
Contributions to OP-TEE are managed by the OP-TEE :ref:`core_team` and anyone
can contribute to OP-TEE as long as it is understood that it will require a
`Signed-off-by` tag from the one submitting the patch(es). The Signed-off-by tag
is a simple line at the end of the explanation for the patch, which certifies
that you wrote it or otherwise have the right to pass it on as an open source
patch (see below). You thereby assure that you have read and are following the
rules stated in the ``Developer Certificate of Origin`` as stated below.

.. _DCO:

Developer Certificate of Origin
*******************************

.. code-block:: none

    Developer Certificate of Origin
    Version 1.1

    Copyright (C) 2004, 2006 The Linux Foundation and its contributors.
    660 York Street, Suite 102,
    San Francisco, CA 94110 USA

    Everyone is permitted to copy and distribute verbatim copies of this
    license document, but changing it is not allowed.


    Developer's Certificate of Origin 1.1

    By making a contribution to this project, I certify that:

    (a) The contribution was created in whole or in part by me and I
        have the right to submit it under the open source license
        indicated in the file; or

    (b) The contribution is based upon previous work that, to the best
        of my knowledge, is covered under an appropriate open source
        license and I have the right under that license to submit that
        work with modifications, whether created in whole or in part
        by me, under the same open source license (unless I am
        permitted to submit under a different license), as indicated
        in the file; or

    (c) The contribution was provided directly to me by some other
        person who certified (a), (b) or (c) and I have not modified
        it.

    (d) I understand and agree that this project and the contribution
        are public and that a record of the contribution (including all
        personal information I submit with it, including my sign-off) is
        maintained indefinitely and may be redistributed consistent with
        this project or the open source license(s) involved.

We have borrowed this procedure from the Linux kernel project to improve
tracking of who did what, and for legal reasons.

To sign-off a patch, just add a line in the commit message saying:

.. code-block:: none

    Signed-off-by: Random J Developer <random@developer.example.org>

Use your real name or on some rare cases a company email address, but we
disallow pseudonyms or anonymous contributions.

GitHub
******
This section describes how to use GitHub for OP-TEE development and
contributions.

Setting up an account
=====================
You do not need to own a GitHub account in order to clone a repository. But if
you want to contribute, you need to create an account at GitHub_. Note that a
free plan is sufficient to collaborate.

SSH is recommended to access your GitHub repositories securely and without
supplying your username and password each time you pull or push something.
To configure SSH for GitHub, please refer to `Connecting to GitHub with SSH`_.

Forking
=======
Only owners of the OP-TEE projects have write permissions to the git
repositories of those projects. Contributors **should fork** ``OP-TEE/*.git``
and/or ``linaro-swg/*.git`` into their own account, then work on this forked
repository. The complete documentation about **forking** can be found at `fork a
repo`_.

Creating pull requests
======================
When you want to submit a patch to the OP-TEE project, you are supposed to
create a `pull request`_ to the git where you forked your git from. How that is
done using GitHub is explained at the GitHub `pull request`_ page.

Commit messages
===============

    - The **subject line** should explain **what** the patch does as precisely
      as possible. It is usually prefixed with keywords indicating which part of
      the code is affected, but not always. Avoid lines longer than 80
      characters.

    - The **commit description** should give more details on **what** is
      changed, and explain **why** it is done. Indication on how to enable and
      use some particular feature can be useful, too. Try to limit line length
      to 72 characters, except when pasting some error message (compiler
      diagnostic etc.). Long lines are allowed to accommodate URLs, too
      (preferably use URLs in a Fixes: or Link: tag).

    - The commit message **must** end with a blank line followed by some tags,
      including your ``Signed-off-by:`` tag. By applying such a tag to your
      commit, you are effectively declaring that your contribution follows the
      terms stated by :ref:`DCO` (in the DCO section there is also a complete
      example).

    - Other tags may be used, such as:

        - ``Tested-by: Test R <test@r.com>``
        - ``Acked-by: Acke R <acke@r.com>``
        - ``Suggested-by: Suggeste R <suggeste@r.com>``
        - ``Reported-by: Reporte R <reporte@r.com>``

    - When citing a previous commit, whether it is in the text body or in a
      Fixes: tag, always use the format shown above (12 hexadecimal digits
      prefix of the commit ``SHA1``, followed by the commit subject in double
      quotes and parentheses).

Review feedback
===============
It is very likely that you will get review comments from other OP-TEE users
asking you to fix certain things etc. When fixing review comments, do:

    - **Add** `fixup` patches on **top** of your existing branch. **Do not**
      squash and force push while fixing review comments.

    - When all comments have been addressed, just write a simple messages in the
      comments field saying something like "All comments have been addressed".
      By doing so you will notify the maintainers that the fix might be ready
      for review again.

Finalizing your contribution
============================
Once you and reviewers have agreed on the patch set, which is when all the
people who have commented on the pull request have given their ``Acked-by:`` or
``Reviewed-by:``, you need to consolidate your commits:

Use ``git rebase -i`` to squash the fixup commits (if any) into the initial
ones. For instance, suppose the ``git log --oneline`` for you contribution looks
as follows when the review process ends:

.. code-block:: none

    <sha1-commit4> [Review] Do something
    <sha1-commit3> [Review] Do something
    <sha1-commit2> Do something else
    <sha1-commit1> Do something

Then you would do:

.. code-block:: bash

    $ git rebase -i <sha1-commit1>^

Edit the commit script so it looks like so:

.. code-block:: none

    pick <sha1-commit1> Do something
    squash <sha1-commit3> [Review] Do something
    squash <sha1-commit4> [Review] Do something
    pick <sha1-commit2> Do something else

Add the proper tags (``Acked-by: ...``, ``Reviewed-by: ...``, ``Tested-by:
...``) to the commit message(s) for each and every commit as provided by the
people who reviewed and/or tested the patches.

.. hint::

    ``git commit --fixup=<sha1-of-commit-to-fix>`` and later on ``git rebase -i
    --autosquash <sha1-of-first-commit-in-patch-serie>^1`` is pretty convenient
    to use when adding review patches and doing the final squash operation.

Once ``rebase -i`` is done, you need to force-push (``-f``) to your GitHub
branch in order to update the pull request page.

.. code-block:: bash

    $ git push -f <my-remote> <my-branch>

After completing this it is the project maintainers job to apply your patches to
the master branch.

.. _fork a repo: https://help.github.com/articles/fork-a-repo
.. _GitHub: https://github.com
.. _Connecting to GitHub with SSH: https://help.github.com/articles/connecting-to-github-with-ssh
.. _pull request: https://help.github.com/articles/creating-a-pull-request

