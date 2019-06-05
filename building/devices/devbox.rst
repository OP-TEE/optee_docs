.. _devbox:

############
DeveloperBox
############

The instructions here will tell how to build OP-TEE for `DeveloperBox`_.

.. _devbox_build_instructions:

Build instructions
******************

1. Follow the ":ref:`get_and_build_the_solution`" in :ref:`build`
   from step 1 to step 3.

2. Initialize EDK2 submodule

    .. code-block:: bash
        :linenos:

        $ cd <optee-project>/edk2
        $ git submodule update --init

3. Follow ":ref:`get_and_build_the_solution`" step 4 & 5

4. Stage a new OP-TEE update capsule. This updates TF-A, OP-TEE and UEFI.

    .. code-block:: bash
        :linenos:

        $ fwupdate --apply {50b94ce5-8b63-4849-8af4-ea479356f0e3} \
          > <optee-project>/edk2-platforms/Build/DeveloperBox/RELEASE_GCC5/FV/\
          > SYNQUACERFIRMWAREUPDATECAPSULEFMPPKCS7.Cap

    .. hint::

        Change ``RELEASE_GCC5`` to ``DEBUG_GCC5`` for debug build.

5. Reboot to update.

6. Follow the rest of":ref:`get_and_build_the_solution`" from step 7


.. _DeveloperBox: https://www.96boards.org/product/developerbox/
