.. _dev-api

Documentation
=================

From v4 onwards, we use  `Read the Docs`_ with Sphinx to build the documentation and host it on
`GitHub <https://github.com/inex/ixp-manager-docs>`_.

.. _Read the Docs: https://docs.readthedocs.org/en/latest/index.html


Building Locally
----------------

If you haven't already, install Sphinx:

::

  pip install sphinx sphinx-autobuild

The documentation can then be built locally as follows:

::

  git clone https://github.com/inex/ixp-manager-docs.git
  cd ixp-manager-docs/docs
  make html
  open _build/html/index.html
