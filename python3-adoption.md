## Frustration

A Python user is starting a project and thinks to themselves,
"Yay, new code! I can use Python 3 for this!".
They install the latest [Anaconda][] for Py3 and get to work.
A few days and hundreds of lines of code later they find out that a particular
library they need (maybe [imposm.parser][imposm]) only supports Python 2.
Our well intentioned user sighs, re-installs Anaconda for Py2,
and carries on.
Maybe next time (or [maybe not][py3readiness]).
(This is a semi-autobiographical story.)

Elsewhere, a Python library maintainer is excited about Py3's
new [asyncio][] module and could put it to immediate use but
doesn't want to alienate users who are stuck on Py2.

----------

There are many valid reasons to be using Py2 today:
a dated dependency, the inertia of existing code,
not wanting to break a working setup,
not knowing how/why to switch, and lack of time.

There are also many valid reasons for wanting to develop exclusively
for Py3: access to [new features][], reduced support burden,
simplified maintenance,
wanting to get ahead of the [2020 end-of-support][python-eol] for Py2,
and lack of time.
These tensions have the potential to create much frustration in
the Python community,
but I think with some intentional effort on the part of Python
developers and leaders it will all be fine.

## A Path to Universal Python 3

At a recent meeting of Python developers I sat down with several people,
including @[Mbussonn][], @[tacaswell][], @[kylebarbary][] and others,
to discuss what we as developers can do to speed the adoption of Py3.
Below are some of the ideas I took note of, you can also read Matthias'
hardline and tongue-in-cheek take [here][matthias].

### Make Python 3 the Default

- Use Py3 in demos, talks, lessons, and documentation.
  If Py3 usage is highly visible that will increase people's willingness
  to adopt it themselves.
  (Software Carpentry's [Python lesson][swc-python] has recently been converted.)
- Make Py3 the default download for [Anaconda][] and [Canopy][].
  [Miniconda][] and [python.org][] already make Py3 downloads
  highly visible.
  Anaconda's [download page][anaconda-dl]
  makes Py3 look like a second-class citizen.
- Right now we often refer to Py2 as "Python" and Py3 as "Python 3".
  Let's flip that.
  Let's make "Anaconda" come with Py3 and "Anaconda2" come with Py2.
  Same for Miniconda.
  We could even go so far as rebranding Py2 as
  ["Legacy Python"][legacy-python].

### Write, Write, Write

To keep the transition to Py3 smooth we should saturate the internet
with useful tutorials about making the conversion.
There are already [many][pyporting], [many][py3porting]
[great pages][pocoo-porting] of advice on the topic,
but the more visible this is the better.
Every time you solve a transition problem you should write about it.
My [cext23][] project is one attempt to help by giving some examples
of supporting Py2 and Py3 when you have C extensions.

### Maybe Drop Python 2 Support

If you make a new library that you think is best implemented without
Py2 support (e.g. to use Py3-only features)
I think it's time you can do that without feeling guilty.
Similarly, if you maintain a library that could benefit from dropping
Py2 support I think it's now okay to do that,
provided you give plenty of advanced notice.
Users can always stick with older, Py2 compatible versions of your library
and forgo upgrades until they adopt Py3.

However, if you can reasonably support both Py2 and Py3 I think that's
a nice thing to do.
(Though when I say Py2 I really only mean Python 2.7.
Python 2.6 has not been maintained [since 2013][py26-eol].)

## Peace

Python 2.7 will reach the [end of it's supported life][python-eol] in 2020.
If you aren't already transitioned or planning your transition
you are running out of time.
The longer you wait the harder the transition will be.
But don't worry, we're here for you.

In a few years this will have all faded into the past and it will be nice
to not have to talk about it anymore.
Maybe then we can fix Python packaging and logging.

[Anaconda]: https://store.continuum.io/cshop/anaconda/
[anaconda-dl]: http://continuum.io/downloads
[Canopy]: https://www.enthought.com/products/canopy/
[Miniconda]: http://conda.pydata.org/miniconda.html
[imposm]: http://imposm.org/docs/imposm.parser/latest/
[py3readiness]: http://py3readiness.org/
[asyncio]: https://docs.python.org/3/library/asyncio.html
[new features]: https://docs.python.org/3/whatsnew/index.html
[Mbussonn]: https://twitter.com/Mbussonn
[tacaswell]: https://twitter.com/tacaswell
[kylebarbary]: https://twitter.com/kylebarbary
[matthias]: http://carreau.github.io/posts/planning-an-early-death-for-python-2.html
[swc-python]: http://swcarpentry.github.io/python-novice-inflammation/
[python.org]: https://www.python.org/downloads/
[legacy-python]: https://twitter.com/jakevdp/status/645288857382944768
[pyporting]: https://docs.python.org/3/howto/pyporting.html
[py3porting]: http://python3porting.com/
[pocoo-porting]: http://lucumr.pocoo.org/2013/5/21/porting-to-python-3-redux/
[cext23]: https://github.com/jiffyclub/cext23
[py26-eol]: https://www.python.org/dev/peps/pep-0361/
[python-eol]: https://www.python.org/dev/peps/pep-0373/#update
