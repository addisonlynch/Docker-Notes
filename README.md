# Docker Notes

[![Build Status](https://travis-ci.org/addisonlynch/Docker-Notes.svg?branch=master)](https://travis-ci.org/addisonlynch/Docker-Notes)

This respository is the source code for addisonlynch.github.io/docker-notes, which are my rudimentary set of usage notes for the [Docker](https://www.docker.com/) containerization platform.

*Warning: these notes have been **in progress** since I began learning Docker. They may contain errors and inconsistencies.*

## Development

These notes are built with the [Sphinx](https://github.com/sphinx-doc/sphinx) documentation engine using [reStructuredText](http://docutils.sourceforge.net/rst.html) as a markup language. With each push to the ``master`` branch of this repository, the documentation is deployed to addisonlynch.github.io/docker-notes using [TravisCI](https://pypi.org/project/doc8/) and the deployment tool [doctr](https://github.com/drdoctr/doctr).

[doc8](https://pypi.org/project/doc8/) is used for linting the RST files. The linter is run with each Travis build.

### Building The Documentation

These notes can be served locally using ``sphinx-autobuild``, using ``make livehtml``.

To build the documentation only, use ``make html``.

### Linting

Use ``doc8 source`` to lint the documentation.


## Dependencies

* ``doc8``
* ``sphinx``
* ``sphinx_rtd_theme``
* ``sphinx-autobuild``


