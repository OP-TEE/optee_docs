# OP-TEE documentation
This is official documentation for the OP-TEE project. Before OP-TEE v3.5.0 it
used to be spread across all different OP-TEE gits making up the OP-TEE project
as well as optee.org. But starting with OP-TEE v3.5.0 we have gathered all
documentation at single place (i.e., this git).

Even though GitHub renders `*.rst` somewhat OK, you are not suppossed to browse
the documentation there/here. Instead you should go to [optee.readthedocs.io],
where you will find the complete documentation rendered using Sphinx.

## Building locally

Set up the necessary environment with sphinx packages installed:

```bash

$ sudo apt-get install python3 python3-virtualenv
$ virtualenv -p /usr/bin/python3 venv
$ . ./venv/bin/activate
$ pip install -r requirements.txt

```

To build html pages from rst files, after
activating venv run from the top directory:

```bash

(venv) $ make html
...
build succeeded, 7 warnings.

The HTML pages are in _build/html.
```

[optee.readthedocs.io]: https://optee.readthedocs.io
