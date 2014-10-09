I have been using [Conda][] (via [Miniconda][]) for managing my Python
development environments and packages for close to a year now,
so I thought I'd write up my thoughts so far for others.

Conda is both an environment manager (an alternative to [virtualenv][])
and an installation tool (an alternative to [pip][]).
You can also use Conda to build your packages and distribute them via
[Binstar][].

So, what does Conda do well and what needs improvement?

# Version

As of this writing I am using Conda version 3.7.1.

# The Good

- Conda installs binary packages.
  There's no compiling extensions, so even installing SciPy goes fast.
  This makes installing SciPy on Windows a breeze.
- Conda caches packages locally for reuse.
  If you make a new environment
  Conda won't have re-download all those packages.
- Conda environments just work.
  New environments are directories on your computer.
  Done with one? Delete the directory.
- Conda and Binstar are a great way to distribute packages.
  If you need to distribute packages that have compiled extensions
  or rely on hard-to-build packages, you can pre-build binaries and
  upload them to [Binstar][] for others to use.
  (Assuming you have access to the same OS your users will use.
  I've heard Continuum is working on a build service that may help with this.)
- If you're running continuous tests on [Travis-CI][] Conda can help
  drastically speed up your build times.
  See the documentation at http://conda.pydata.org/docs/travis.html.
  I also use Conda for provisioning web servers.

# The Annoying

- Conda environments are very [bash][] centric.
  If you use a different shell (like [fish][]) you may need to
  [hack together your own Conda environment tools][fish-conda].
- Conda packages are sometimes behind the latest releases.
  Management of Conda packages seems to be pretty manual. It often takes
  a long time for them to be updated after new releases, especially
  for the less commonly used libraries.
- Conda doesn't quite replace [pip][].
  There aren't Conda packages for every Python package,
  and realistically there can't be.
  When you need a package that's not on Conda it's fine to use pip.
  But this will happen in literally every environment you make,
  so it's kind of silly that pip isn't installed by default.
  (You can [make it so pip is installed by default][conda-config].)

# The Bad

I note above that Conda can't install every package and you sometimes
need to fall back to pip.
Well, it turns out that it is impossible to install some packages
into Conda environments via pip (or even `python setup.py install`).
The problem arises from Conda's treatment of C libraries.
When you install a Python library that links to a C library installed
by Conda, that Python library will not be able to find the C library
at runtime.
See lots of discussion on these GitHub issues:
https://github.com/conda/conda-recipes/issues/110 and
https://github.com/ContinuumIO/anaconda-issues/issues/177.
(These problems seem to be most egregious on Mac OS X.)

The Python packages that are known to fail all work with SSL.
These include [cryptography][] and [psycopg][].
(Continuum has recently added a `cryptography` package to Conda
to help alleviate this problem, but not `psycopg`.)

The workarounds for this problem include setting sketchy variables
like `DYLD_LIBRARY_PATH` or building the packages yourself via Conda,
which is a multi-step process.
Neither is beginner friendly.

# Summary

I love and will continue to use [Conda][],
and I will continue to recommend [Anaconda][] to people
who are interested in getting started with scientific Python.
But there remains a gap between Conda and Python's standard
build systems that is at best inconvenient and at worst impassable
(unless you possess the experience required to navigate the workarounds).
People who are expecting to use [psycopg][] (and likely some other libraries)
should be aware that Conda is not going to make their lives easier.

I think it would be helpful for users if Conda could seamlessly fall
back on [PyPI][] packages when Conda doesn't have something pre-built.
Need `psycopg2`?
Maybe a `conda install --pypi psycopg2` could download `psycopg2` from
PyPI and build it appropriately so that it will work seamlessly with Conda
environments.

Given the state of Conda as a relatively new tool I'm optimistic
that the developers will be able to improve it for everyone given more time.
I use Conda literally almost every day and I thank the developers for it.

[Conda]: http://conda.pydata.org/
[Miniconda]: http://conda.pydata.org/miniconda.html
[Anaconda]: http://docs.continuum.io/anaconda/index.html
[virtualenv]: http://virtualenv.readthedocs.org/en/latest/
[pip]: https://pip.pypa.io/en/latest/
[Binstar]: https://binstar.org/
[bash]: http://www.gnu.org/software/bash/
[fish]: http://fishshell.com/
[fish-conda]: https://gist.github.com/jiffyclub/9679788
[conda-config]: http://conda.pydata.org/docs/config.html
[cryptography]: https://cryptography.io/en/latest/
[psycopg]: http://initd.org/psycopg/
[Travis-CI]: https://travis-ci.org/
[Continuum]: http://continuum.io/
[PyPI]: https://pypi.python.org/pypi
