## Way back...

About [a year ago][ipb-ann], inspired by [Greg Wilson][], I wrote
[ipythonblocks][] as a fun way for students (and anyone else!) to practice
writing Python with immediate, step-by-step, visual feedback about what their
code is doing. When I've taught using ipythonblocks it has always been a
hit--people love making things they can see. And after making things people
love to share them.

Sometime last year [Tracy Teal][] suggested I make a site where students could
post their work from ipythonblocks, share it, and even grab the work of others
to remix. Today I'm happy to announce that that site is live:
[ipythonblocks.org][].

## How it works

With the [latest release of ipythonblocks][ipb-pypi] students can use `post_to_web`
and `from_web` methods to interact with ipythonblocks.org. `post_to_web` can
include code cells from the notebook so the creation process can be shared, not
just the final result. `from_web` can pull a grid from ipythonblocks.org for
a student to remix locally. See [this notebook][ipborg-demo] for a demonstration.

## Thank you

There are many people to thank for helping to make [ipythonblocks.org][] possible.
Thanks to Tracy Teal for the original idea, thanks to [Rackspace][] and
[Jesse Noller][] for providing hosting, and thanks to [Kyle Kelley][] for
helping with ops and deployment. Most of all, thanks to my family for putting
up with me working at a startup *and* taking on projects.

[ipb-ann]: http://penandpants.com/2013/01/10/ipythonblocks-a-visual-tool-for-practicing-python/
[Greg Wilson]: http://third-bit.com/
[ipythonblocks]: https://github.com/jiffyclub/ipythonblocks
[Tracy Teal]: http://idyll.org/~tracyt/
[ipythonblocks.org]: http://ipythonblocks.org
[ipb-pypi]: https://pypi.python.org/pypi/ipythonblocks
[ipborg-demo]: http://nbviewer.ipython.org/github/jiffyclub/ipythonblocks/blob/master/demos/ipythonblocks_org_demo.ipynb
[Rackspace]: http://www.rackspace.com/
[Jesse Noller]: http://jessenoller.com/
[Kyle Kelley]: https://twitter.com/rgbkrk
