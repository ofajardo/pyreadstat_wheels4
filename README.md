# pyreadstat_wheels4
Wheels for pyreadstat

This is a repo building wheels for pyreadstat using the multibuild package. 
It is a modification from the older repo https://github.com/ofajardo/pyreadstat_wheels3.
The main modificaiton here is that we are using the original multibuild repo, and doing some
modifications to be able to build for mac M1.
Previously from 2 to 3 the modification was that here we use azure to build linux and
mac instead of Travis. Numpy wheels was taking as model for building the azure-pipelines.yml
and azure templates, see [here](https://github.com/MacPython/numpy-wheels)

Wheels are uploaded to Anaconda Cloud. In order to do that a anaconda cloud account had 
to be set and the token retrived via WEB UI 
(also possible with CLI), set the Token in Azure and APPVEYOR as secret environment variable
for the project, and then just upload the wheels using the Anaconda-client in azure-pipelines.yml and
appveyor.yml. 

The wheels are then visible in https://anaconda.org/ofajardo/pyreadstat/files


########################################
Building and uploading pyreadstat wheels
########################################

We automate wheel building using this custom github repository that builds on
the azure-pipelines OSX machines, azure-pipelines Linux machines, and the Appveyor VMs.


How it works
============

The wheel-building repository:

* does a fresh build of any required C / C++ libraries;
* builds a pyreadstat wheel, linking against these fresh builds;
* processes the wheel using delocate_ (OSX) or auditwheel_ ``repair``
  (Manylinux1_).  ``delocate`` and ``auditwheel`` copy the required dynamic
  libraries into the wheel and relinks the extension modules against the
  copied libraries;
* uploads the built wheels to a Rackspace container kindly donated by Rackspace
  to scikit-learn.

The resulting wheels are therefore self-contained and do not need any external
dynamic libraries apart from those provided as standard by OSX / Linux as
defined by the manylinux1 standard.


Triggering a build
==================

You will likely want to edit the ``azure-pipelines.yml`` and ``appveyor.yml`` files to
specify the ``BUILD_COMMIT`` before triggering a build - see below.

You can trigger a build by:

* making a commit to the `pyreadstat-wheels` repository (e.g. with `git
  commit --allow-empty`); or
* clicking on the circular arrow icon towards the top right of the azure-pipelines
  page, to rerun the previous build.

In general, it is better to trigger a build with a commit, because this makes
a new set of build products and logs, keeping the old ones for reference.
Keeping the old build logs helps us keep track of previous problems and
successful builds.

Which pyreadstat commit does the repository build?
==================================================

The ``pyreadstat-wheels`` repository will build the commit specified in the
``BUILD_COMMIT`` at the top of the ``azure-pipelines.yml`` and ``appveyor.yml`` files.
This can be any naming of a commit, including branch name, tag name or commit
hash.

Note: when making a release, it's best to only push the commit (not the tag) of
the release to the ``pyreadstat`` repo, then change ``BUILD_COMMIT`` to the
commit hash, and only after all wheel builds completed successfully push the
release tag to the repo.  This avoids having to move or delete the tag in case
of an unexpected build/test issue.

Uploading the built wheels to pypi
==================================

Be careful, these links point to containers on a distributed content delivery
network.  It can take up to 15 minutes for the new wheel file to get updated
into the containers at the links above.

When the wheels are updated, you can download them to your machine manually,
and then upload them manually to pypi, or by using twine_.  You can also use a
script for doing this, housed at :
https://github.com/MacPython/terryfy/blob/master/wheel-uploader
When the wheels are updated, you can download them to your machine manually,
and then upload them manually to pypi, or by using twine_.  You can also use a
script for doing this, housed at :
https://github.com/MacPython/terryfy/blob/master/wheel-uploader

For the ``wheel-uploader`` script, you'll need twine and `beautiful soup 4
<bs4>`_.

You will typically have a directory on your machine where you store wheels,
called a `wheelhouse`.   The typical call for `wheel-uploader` would then
be something like::

    VERSION=0.2.0
    CDN_URL=https://3f23b170c54c2533c070-1c8a9b3114517dc5fe17b7c3f8c63a43.ssl.cf2.rackcdn.com
    wheel-uploader -u $CDN_URL -s -v -w ~/wheelhouse -t all pyreadstat $VERSION

where:

* ``-u`` gives the URL from which to fetch the wheels, here the https address,
  for some extra security;
* ``-s`` causes twine to sign the wheels with your GPG key;
* ``-v`` means give verbose messages;
* ``-w ~/wheelhouse`` means download the wheels from to the local directory
  ``~/wheelhouse``.

``pyreadstat`` is the root name of the wheel(s) to download / upload, and
``0.2.0`` is the version to download / upload.

In order to upload the wheels, you will need something like this
in your ``~/.pypirc`` file::

    [distutils]
    index-servers =
        pypi

    [pypi]
    username:your_user_name
    password:your_password

So, in this case, `wheel-uploader` will download all wheels starting with
`pyreadstat-0.2.0-` from the URL in ``$CDN_URL`` above to ``~/wheelhouse``, then
upload them to PyPI.

Of course, you will need permissions to upload to PyPI, for this to work.

.. _manylinux1: https://www.python.org/dev/peps/pep-0513
.. _twine: https://pypi.python.org/pypi/twine
.. _bs4: https://pypi.python.org/pypi/beautifulsoup4
.. _delocate: https://pypi.python.org/pypi/delocate
.. _auditwheel: https://pypi.python.org/pypi/auditwheel
