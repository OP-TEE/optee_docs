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
One can purchase the `GlobalPlatform Compliance Test suite`_ which is
GlobalPlatforms own test suite for testing TEE implementations adhering to the
GlobalPlatforms interfaces.

.. hint::
        Members of GlobalPlatform can download this for free at the
        GlobalPlatform members pages. This something that the OP-TEE project
        **cannot** help with. If you need help with that, please reach out to
        the liason at GlobalPlatform.

xtest can be extended/patched to include the GlobalPlatform Compliance Test
suite. This can be done by downloading the GlobalPlatform Compliance Test suite
(a ``*.7z`` file) and add an additional compiler flag (``GP_PACKAGE``) to
the ``make`` invocation line, example:

.. code-block:: bash

	$ make GP_PACKAGE=/tmp/TEE_Initial_Configuration-Test_Suite_v2_0_0_2-2017_06_09.7z

.. note::
        Starting from OP-TEE v3.11.0, OP-TEE was updated to support the
        ``TEE_Initial_Configuration-Test_Suite_v2_0_0_2-2017_06_09.7z`` version
        from the GlobalPlatform Compliance Test suite. That is the only
        supported version after OP-TEE v3.11.0. If you need to run an earlier
        version of the GlobalPlatform Compliance Test suite then you need to
        follow the instructions in the documentation for OP-TEE v3.9.0 and
        earlier.


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

The :ref:`optee_os` repository is required to run the checks. It's location may
be passed using the `OPTEE_OS_PATH` environment variable:

.. code-block:: bash

  export OPTEE_OS_PATH=/path/to/optee_os

In case `OPTEE_OS_PATH` is unset or empty, the dispatcher script will default to `../optee_os`.

.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
.. _GlobalPlatform Compliance Test suite: https://store.globalplatform.org/product/tee-initial-configuration-test-suite-with-excluded-tests-list-v2-0-0-2/
.. _GPL-2.0: http://opensource.org/licenses/GPL-2.0
