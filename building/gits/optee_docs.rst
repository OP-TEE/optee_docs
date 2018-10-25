.. _optee_docs:

##########
optee_docs
##########
This is the Git where all official OP-TEE documentation resides and this is what
you are reading right now. Here we will give instructions on how to write and
build the documentation as well as give some guidelines on what to do and not to
do. Note that the documentation is written for Sphinx_. So, even though GitHub
for example renders ``*.rst`` files somewhat OK, that is still not the preferred
way to read and view the documentation. Instead head over to
https://optee.readthedocs.io where the final output is stored and nicely
rendered using Sphinx.

git location
************
https://github.com/OP-TEE/optee_docs

Install Sphinx
**************
Before doing anything else, first install Sphinx and the dependencies.

.. code-block:: bash

    $ sudo apt install graphviz python3-sphinx python3-sphinx-rtd-theme

Build optee_docs
****************
.. code-block:: bash

    $ git clone https://github.com/OP-TEE/optee_docs
    $ cd optee_docs
    $ make html

After this step all documentation should have been built and you can open
``<optee_docs>/_build/html/index.html`` in your browser to see the result and
browse the documentation.

.. hint::

    By using a Linux tool called ``entr``. You can automatically rebuild the
    pages your are working with. First get the package ``$ sudo apt install
    entr``, then:

    .. code-block:: bash

        $ cd <optee_docs>
        $ find . -name "*.rst" | entr -c make html

    With this, ``entr`` will automatically rebuild the documentation everytime
    you make change and save a file. Which means you only have to save the file
    in your editor and refresh the browser page to see the changes locally.

General guidelines
******************

Linking
=======

Internal links
--------------
Internally within a Sphinx project you can link various pages by referring to a
keyword specified right above a section, chapter or subsection. This means that
you don't have to make hardlinks to certain files. Instead Sphinx will just
figure out where it is for you. Example I have to files, file `compiler.rst` and
`toolchain.rst`. They could look like this:

compiler.rst example
^^^^^^^^^^^^^^^^^^^^
.. code-block:: rst
    :linenos:
    :emphasize-lines: 6, 8

    ########
    Compiler
    ########
    Bla bla bla

    .. _compiler_flags:

    Compiler Flags
    **************

toolchain.rst example
^^^^^^^^^^^^^^^^^^^^^
.. code-block:: rst
    :linenos:
    :emphasize-lines: 5

    ########
    Toolchain
    ########
    Bla bla bla to see find out more about various flags, please refer
    :ref:`compiler_flags`.


What we can see in the example, is that on line 5 in ``toolchain.rst`` we refer
to the keyword in ``compiler.rst`` by using ``:ref:`compiler_flags```. This
would render a direct link to that section in ``compiler.rst``.

General recommendation for OP-TEE internal linking
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    - Things about general things doesn't have to be prefixed with the "document
      name".

    - Things that are specific should be prefixed with the "document name".

Example: the "Contact" section is generic so it's there is no need for prefix.
But for example HiKey 620 build instructions are specific to HiKey 620, so there
we shall prefix keyword for internal linking.

rst files
---------
The rst files should have descriptive names, but even more important is where
you decide to put the files. Even though it's not a problem to move files
around, we have to remember that we tend to quite often give links to
documentation from at GitHub, emails etc. If we move files, there is a high
likelihood that they will become dead links in the future (404's). So think
twice before adding a new file or moving an existing file.

Sections, chapters
------------------
We have adopted the Sphinx recommended way of using sections, chapters,
subsections etc, those are:

    - # with overline, for parts
    - \* with overline, for chapters
    - =, for sections
    - \-, for subsections
    - ^, for subsubsections
    - ", for paragraphs


.. _Sphinx: http://www.sphinx-doc.org
