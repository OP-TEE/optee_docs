.. _optee_benchmark:

###############
optee_benchmark
###############
This page describes how to get and build the OP-TEE benchmark framework. For the
architectural details, please refer to the :ref:`benchmark_framework` page
instead.

git location
************
https://github.com/linaro-swg/optee_benchmark

License
*******
.. todo::

    Joakim: Necessary to state that here? Changing the "License headers" page to
    instead become a "License" page and add addtional sections.

The software is provided under the `BSD 2-Clause`_ license.

Build instructions
******************
The benchmark framework spans across different architectural layers and
therefore it doesn't make much sense to build it as a standalone build.
Therefore we only give guidance telling how to enable it in full OP-TEE
developer builds. For general instructions for full OP-TEE developer builds,
please refer to instructions at the :ref:`build` page. But otherwise follow the
instructions below to enable the benchmark framework.

Enable the benchmark framework
==============================
Before using Benchmark framework, OP-TEE should be rebuilt with the
``CFG_TEE_BENCHMARK`` flag enabled so that the benchmark framework will be
enabled in all architectural layers. You do that by:

.. code-block:: bash

    $ cd <optee-project>/build
    $ make CFG_TEE_BENCHMARK=y

Benchmark application usage
===========================
When everything has been built (flashed) and you have booted up the device and
you have a console ready to accept command, then the next step is to run the
actual benchmark application together with the host/TA application you intend to
benchmark. You do this my giving the host applicant as argument to the
optee_benchmark binary. Let's say for example that you intend to benchmark the
:ref:`hello_world` example. Then you invoke the benchmark like this:

.. code-block:: bash

    $ benchmark hello_world

When client_app finish the execution, optee_benchmark will generate
``<client_app>.ts`` time stamp data file in the same directory, where Client
Application is stored (i.e., relative to `hello_world` in this case).

.. _BSD 2-Clause: http://opensource.org/licenses/BSD-2-Clause
