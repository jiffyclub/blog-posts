These instructions detail how I install the scientific Python stack on my
Mac. I'm running the latest OS X Mountain Lion but I think these instructions
should work back to Snow Leopard. These instructions differ from my
[previous set][old-instr] primarily in that I now use [Homebrew][] to
install [NumPy][], [SciPy][], and [matplotlib][]. I do this because Homebrew
makes it easier to compile these with non-standard options that work around
an [issue with SciPy on OS X][scipy-issue].

I cover here how to install Python and the basic scientific Python stack:

- [NumPy][]
- [SciPy][]
- [matplotlib][]
- [IPython][]
- [pandas][]

If you need other libraries they most likely can be installed via [pip][] and
any dependencies can probably be installed via [Homebrew][].

# Command Line Tools

The first order of business is to install the Apple command line tools. These
include important things like development headers, `gcc`, and `git`. Head
over to [developer.apple.com/downloads][apple-dev], register for a free
account, and download (then install) the latest "Command Line Tools for Xcode"
for your version of OS X.

If you've already installed [Xcode][] on Lion or Mountain Lion then you can
install the command line tools from the preferences. If you've installed
Xcode on Snow Leopard then you already have the command line tools.

# Homebrew

[Homebrew][] is my favorite package manager for OS X. It builds packages from
source, intelligently re-uses libraries that are already part of OS X, and
encourages best practices like installing Python packages with [pip][].

To install Homebrew paste the following in a terminal:

    ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"

The `brew` command and binaries it installs will go in the directory
`/usr/bin/local` so you want to make sure that goes at the front of your
system's `PATH`. As long as you're at it, you can also add the directory where
Python scripts get installed. Add the following line to your `.profile`,
`.bash_profile`, or `.bashrc` file:

    export PATH=/usr/bin/local:/usr/bin/local/share/python:$PATH

At this point you should close your terminal and open a new one so that this
`PATH` setting is in effect for the rest of the installation.

# Python

Now you can use `brew` to install Python:

    brew install python

Afterwards you should be able to run the commands

    which python
    which pip

and see

    /usr/local/bin/python
    /usr/local/bin/pip

for each, respectively. (It's also possible to install Python 3 using
Homebrew: `brew install python3`.)

# NumPy

It is possible to use `pip` to install [NumPy][], but I use a Homebrew recipe
so I avoid some problems with SciPy. The recipe isn't included in stock
Homebrew though, it requires "tapping" two other sources of Homebrew formula:

    brew tap homebrew/science
    brew tap samueljohn/python

You can learn more about these at their respective repositories:

* [homebrew/science][homebrew-science]
* [samueljohn/python][samueljohn-python]

With those repos tapped you can install NumPy. I install it against
[OpenBLAS][] to avoid [a SciPy issue][scipy-issue]:

    brew install numpy --with-openblas

# SciPy

[SciPy][] requires gfortran, which can be installed by Homebrew:

    brew install gfortran
    brew install scipy --with-openblas

# matplotlib

[matplotlib][] generally installs just fine via `pip` but the custom Homebrew
formula takes care of installing optional dependencies too:

    brew install matplotlib

# IPython

You'll want [Notebook][] support with [IPython][] and that requires some extra
dependencies, including [ZeroMQ][] via `brew`:

    brew install zeromq
    pip install jinja2
    pip install tornado
    pip install pyzmq
    pip install ipython

# pandas

[Pandas][pandas] should install via `pip`:

    pip install pandas

# Testing It Out

The most basic test you can do to make sure everything worked is open up
an IPython  session and type in the following:

    import numpy
    import scipy
    import matplotlib
    import pandas

If there are no errors then you're ready to get started!
Congratulations and enjoy!

[old-instr]: http://penandpants.com/2012/02/24/install-python/
[Homebrew]: http://brew.sh/
[NumPy]: http://www.numpy.org/
[SciPy]: http://scipy.org
[matplotlib]: http://matplotlib.org
[IPython]: http://ipython.org
[pandas]: http://pandas.pydata.org
[scipy-issue]: https://github.com/samueljohn/homebrew-python/issues/12
[apple-dev]: http://developer.apple.com/downloads
[Xcode]: https://developer.apple.com/xcode/
[pip]: https://pypi.python.org/pypi/pip
[homebrew-science]: https://github.com/Homebrew/homebrew-science
[samueljohn-python]: https://github.com/samueljohn/homebrew-python
[OpenBLAS]: http://xianyi.github.com/OpenBLAS/
[ZeroMQ]: http://www.zeromq.org/
[Notebook]: http://ipython.org/notebook.html
