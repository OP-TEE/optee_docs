.. _versal2:

################
AMD Versal Gen 2
################
The following instructions demonstrate how to obtain the OP-TEE source code and build it for the AMD Versal Gen 2 platform.

Supported boards
****************
AMD Versal Gen 2 adaptive SoCs combine world-class programmable logic from AMD with a new high-performance processing system of integrated Arm CPU.

Stay tuned for more updates!

Configuring and building
************************

.. code-block:: bash

	$ mkdir -p optee_project
	$ cd optee_project
	$ repo init -u https://github.com/OP-TEE/manifest.git -m versal2.xml
	$ repo sync

Compilation
***********
Run the following commands to compile all the necessary source code.

.. code-block:: bash

        $ cd build
        $ make toolchains
        $ make all

Build Artifacts
***************
All the build artifacts are available in the output directory.

.. code-block:: bash

	$ cd ../out/bin/

Stay tuned for more updates!
