.. _prerequisites:

#############
Prerequisites
#############
We believe that you can use any Linux distribution to build OP-TEE, but as
maintainers of OP-TEE we are mainly using Ubuntu-based distributions and to be
able to build and run OP-TEE there are a few packages that needs to be
available. Hereafter we provide Docker files which may be used as a reference.

.. tabs::

    .. tab:: Ubuntu 22.04

        .. include:: Dockerfile.Ubuntu-22.04
           :code: text

    .. tab:: Ubuntu 20.04

        .. include:: Dockerfile.Ubuntu-20.04
           :code: text

    .. tab:: Fedora 42

        .. include:: Dockerfile.Fedora-42
           :code: text

    .. tab:: Older

        .. note::

            No longer supported by the OP-TEE community!

        Due to all changes over the years with different names of Python
        packages and different requirement in time for Python2 and/or Python3
        packages, it's not really possible to build more recent versions of
        OP-TEE with something that is older than Ubuntu 18.04. If you for some
        reason need to rebuild OP-TEE using a very old distro, then the best
        strategy for doing is is to check an earlier version of this
        documentation and start with the build instructions from there.
