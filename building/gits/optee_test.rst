.. _optee_test:

##########
optee_test
##########
The optee_test.git contains the source code for the TEE sanity test suite in
Linux using the ARM(R) TrustZone(R) technology. It is typically referred to as
`xtest`. By default there are several thousands of tests when running the code
that is in the git only. However, it is also possible to incorporate tests
coming from GlobalPlatform (see :ref:`globalplatform_tests`). We typically refer
to these to as:

    - **Standard tests**: These are the test that are included in optee_test.
      They are free and open source.

    - **Extended tests**: Those are the tests that are written directly by
      GlobalPlatform. They are **not** open source and they are **not** freely
      available (it's free to members of GlobalPlatform and can otherwise be
      purchased directly from GlobalPlatform).

git location
************
https://github.com/OP-TEE/optee_test

License
*******
.. todo::

    Joakim: Necessary to state that here? Changing the "License headers" page to
    instead become a "License" page and add addtional sections.

The client applications (``optee_test/host/*``) are provided under the
`GPL-2.0`_ license and the user Trusted Applications (``optee_test/ta/*``) are
provided under the `BSD 2-Clause`_.

Build instructions
******************
At the moment you can **only** build the code in this git as part of the entire
system, i.e. as a part of a full OP-TEE developer setup. So, please refer to
the instructions at the :ref:`build` page to learn how to build a full OP-TEE
developer setup. Building purely standalone is **not** possible (*) because:

    - the host code (``xtest``) have dependencies to the :ref:`optee_client` (it
      links against ``libteec``, ``openssl`` and uses various headers)

    - the Trusted Applications have dependencies to the TA-devkit built by
      :ref:`optee_os`.

.. note::

        (*) It is of course possible to build this without a full OP-TEE
        developer setup, but it will require a lot of tweaking with paths, flags
        etc. I.e., one would need to do exactly the same as the full OP-TEE
        developer setup does under the hood.

.. _globalplatform_tests:

Extended test (GlobalPlatform tests)
************************************
One can purchase the `GlobalPlatform Compliance Test suite`_ which comes with
.xml files describing the tests and the Trusted Applications. The standard tests
(xtest + TA's) that are free and open source can be extended to also include the
GlobalPlatform test suite. This is done by:

    - Install the GlobalPlatform ``xml`` files in ``$CFG_GP_PACKAGE_PATH``.

    - Run ``make patch`` (or call make ``xtest-patch`` from the ``build``
      repository) before compiling xtest. This must be run a single time after the
      installation of OP-TEE.

This will:

    - Create new Trusted Applications, that can be found in ``ta/GP_xxx``
    - Create new tests in ``host/xtest``, as for example ``xtest_9000.c``
    - Patches ``xtest_7000.c``, adding new tests.

Then the tests must be compiled with ``CFG_GP_PACKAGE_PATH=<path>``.

It makes use of the following environment variable:

    - ``COMPILE_NS_USER``: ``32`` or ``64`` if application shall be compiled in
      32 bits mode on in 64 bits mode. If ``COMPILE_NS_USER`` is not specified,
      build relies on ``CFG_ARM32_core=y`` from OP-TEE core build to assume
      applications are in 32 bits mode, Otherwise, 64 bits mode is assumed.

.. _optee_test_run_xtest:

Run xtest
*********
It's important to understand that you run ``xtest`` on the device itself, i.e.,
this is nothing that you run on the host machine.

xtest - default
===============
The most simple case is to run the default configuration:

.. code-block:: bash

	$ xtest

xtest - all
===========
This runs all tests within the standard xtest. Using the ``-l`` parameter you
can tweak the amount of tests you will run. ``15`` is the most and ``0`` is the
least.

.. code-block:: bash

	$ xtest -l 15

xtest - single
==============
To run a single test case, just specify its numbers, for example:

.. code-block:: bash

	$ xtest 1001

xtest - family
==============
To run a family (``1xxx``, ``2xxx`` and so on), just specify its number prefixed
with an underscore. This for example will run the 1xxx family.

.. code-block:: bash

	$ xtest _1

xtest - benchmark
=================
To run the benchmark tests, run xtest like this:

.. code-block:: bash

	$ xtest -t benchmark

Here it is also possible to state a number for a certain benchmark test, for
example:

.. code-block:: bash

	$ xtest -t benchmark 2001

xtest - regression
==================
To run the regression tests, run xtest like this:

.. code-block:: bash

	$ xtest -t regression

Here it is also possible to state a number for a certain regression test, for
example:

.. code-block:: bash

	$ xtest -t regression 2004

xtest - aes-perf
================
This is benchmark test for AES and you run it like this:

.. code-block:: bash

	$ xtest --aes-perf

.. note::

    There is an individual help for ``--aes-perf``, i.e.

    ``$ xtest --aes-perf -h``

xtest - sha-perf
================
This is benchmark test for SHA-xxx and you run it like this:

.. code-block:: bash

	$ xtest --sha-perf

.. note::

    There is an individual help for ``--sha-perf``, i.e.

    ``$ xtest --sha-perf -h``

    There you can select other SHA algorithms etc.

.. todo::

    Joakim: Should have a section about --install-ta also.

Coding standards
****************
See :ref:`coding_standards`.

.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
.. _GlobalPlatform Compliance Test suite: https://store.globalplatform.org/product/tee-initial-configuration-test-suite-with-excluded-tests-list-2/
.. _GPL-2.0: http://opensource.org/licenses/GPL-2.0
