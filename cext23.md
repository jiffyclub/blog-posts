One of my upcoming tasks at work is converting [Pandana][] to support both
Python 2 and 3.
The tricky bit is that Pandana has a C extension written in plain C using
the Python 2 C-API, which is [not compatible][cporting] with Python 3.

It seems like the best way to have a C extension that supports both
Python 2 and 3 is to not write the extension in C.
These days there are a [number of alternatives][ppug-ext] that allow you
to write interfaces in Python or something like Python ([Cython][]).
I decided to make a sample project with some C functions to wrap so
that I could try out [CFFI][], [Cython][], and the standard library
[ctypes][] module.

You can find the project with examples of all three and a longer writeup at
https://github.com/jiffyclub/cext23.
Pull requests are welcome on the repo with further examples!

[Pandana]: https://github.com/UDST/pandana
[cporting]: https://docs.python.org/3/howto/cporting.html
[ppug-ext]: https://packaging.python.org/en/latest/extensions/
[Cython]: http://cython.org/
[CFFI]: http://cffi.readthedocs.org/
[ctypes]: https://docs.python.org/3/library/ctypes.html
