My posts on installing Python on Mac OS X seem to be the most popular things
I've posted so I thought I should gather them in one place. Here are some of
your options (by no means all of them), listed from most basic to most advanced.

# All-in-One

There are multiple packaged installers for scientific Python including
[EPD Free][] from [Enthought][] and [Anaconda][] from [Continuum Analytics][].
Of those two I prefer the Anaconda installation because it contains more
packages and because it installs into a single directory in your home folder.
You will have to use the terminal a little to install Anaconda.

To get started download the Mac OS X installer from
[continuum.io/downloads.html][anaconda-dl]. Open a terminal, `cd` to the
folder where Anaconda downloaded, and run the following command:

    bash Anaconda-1.x.x-MacOSX-x86_64.sh

Be sure to replace the `x.x` in the above command with the version number of
Anaconda you downloaded. The install script will ask you some questions, I
recommend taking the defaults (except when you accept the license terms).

Once the installation is complete you need to add Anaconda's directory of
executables to your system's `PATH` environment variable. Add the following
line to your `.profile`, `.bash_profile`, or `.bashrc`:

    export PATH=$HOME/anaconda/bin:$PATH

Open a new terminal once that's in place. Typing `which python` should show
the one in `anaconda/bin`. If you need additional Python packages not included
with Anaconda they can probably be installed using [pip][], which comes with
Anaconda.

# From Source

I expect [Anaconda][] should meet the needs of most people. I'd recommend
going the route of compiling everything from source only if you expect to need
to install things like development versions of software, especially using
[virtualenv][] (and [virtualenvwrapper][], of course).

See my post [Install Scientific Python on Mac OS X][source-post] for
instructions on installing Python and the scientific Python stack from source.
This is how I install Python on my Mac.

[EPD Free]: http://www.enthought.com/products/epd_free.php
[Enthought]: http://www.enthought.com/
[Anaconda]: https://store.continuum.io/cshop/anaconda
[Continuum Analytics]: http://www.continuum.io/
[anaconda-dl]: http://continuum.io/downloads.html
[anaconda-inst]: http://docs.continuum.io/anaconda/install.html#mac-install
[pip]: https://pypi.python.org/pypi/pip
[virtualenv]: http://www.virtualenv.org/en/latest/
[virtualenvwrapper]: https://bitbucket.org/dhellmann/virtualenvwrapper
[source-post]: http://penandpants.com/2013/04/04/install-scientific-python-on-mac-os-x/
