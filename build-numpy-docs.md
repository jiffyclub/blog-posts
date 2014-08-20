This week [docs.scipy.org][] has been down,
but folks still need their [NumPy][] and [SciPy][] docs.
To fill the gap until [docs.scipy.org][] is back up I built
the docs for only the latest stable releases
and uploaded them to GitHub pages:

- [NumPy](http://jiffyclub.github.io/numpy/)
- [SciPy](http://jiffyclub.github.io/scipy/)

How to Build
============

(Note that I'm working on a Mac and these instructions are a little
Mac/Linux oriented. The procedure on Windows would not be drastically
different, though.)

## Install Dependencies

You need several things [installed](http://penandpants.com/install-python/)
in your Python environment in order to build the docs.
I use [conda][] and all of these are readily available through that.
(And as part of [Anaconda][].)
Here's the list of things you'll need:

- [NumPy][]
- [SciPy][]
- [matplotlib][]
- [Sphinx][] (actually builds the docs)
- [docutils][] (dependency of Sphinx)
- [numpydoc][] (adds support for NumPy's docstring format)

[Sphinx][] is a wonderful tool for building static sites from
[reStructedText][] files.
The sites don't necessarily have to have anything to do with Python,
but the real magic of Sphinx is its ability to import Python modules,
read the docstrings from them, and turn those into API documents.
(Read more on Sphinx's [autodoc][] extension.)

## Get the Source Code

Clone the [NumPy][numpy-gh] and [SciPy][scipy-gh] repositories from GitHub.
The source files for the documentation is included with each repository.
You can optionally check out a release tag of the repo so that you're
building the docs for that release. For example, [conda][] had installed
NumPy version 1.8.2 so I wanted to build the docs for that version.
The NumPy repo included a v1.8.2 tag so I checked that out so I'd be building
the docs for that version:

    cd numpy
    git checkout v1.8.2

It's best to have the repo match the version of NumPy you have installed.
If you've just installed NumPy from the GitHub repo you can skip the
checkout of the release tag.

Building the docs depends on some things that are included in the repos
via [Git submodules](http://git-scm.com/book/en/Git-Tools-Submodules).
These need to be pulled in with an extra step. After cloning, run these
commands:

    git submodule init
    git submodule update

That will pull in some extra stuff required for the docs.

## Run the Build

Change directories to the `doc` directory:

    cd doc

You should see a `Makefile` here, and a `source` directory.
That `source` directory contains the [Sphinx][] configuration for
NumPy and all of the source `.rst` files that will be compiled to build
the docs. With all the dependencies installed and from the doc directory
you should be able to use the `Makefile` to start the doc build:

    make html

This will take a while, print out tons of errors and warnings
(that you can hopefully ignore), and finally produce compiled HTML
under the `build/html` directory.
Open `build/html/index.html` to see the result!

## Recipe

If all you want is the commands, here they are
(assuming dependencies are installed):

    git clone https://github.com/numpy/numpy.git
    cd numpy
    git submodule init
    git submodule update
    cd doc
    make html

[docs.scipy.org]: http://docs.scipy.org
[NumPy]: http://www.numpy.org/
[SciPy]: http://scipy.org
[conda]: http://conda.pydata.org/
[Anaconda]: https://store.continuum.io/cshop/anaconda/
[matplotlib]: http://matplotlib.org/
[numpydoc]: https://github.com/numpy/numpydoc
[Sphinx]: http://sphinx-doc.org/
[docutils]: http://docutils.sourceforge.net/
[reStructedText]: http://docutils.sourceforge.net/rst.html
[autodoc]: http://sphinx-doc.org/ext/autodoc.html
[numpy-gh]: https://github.com/numpy/numpy
[scipy-gh]: https://github.com/scipy/scipy
