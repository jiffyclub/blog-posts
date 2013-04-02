[PyCon 2013][] was my first PyCon and it was, bar none, the best conference
I've ever been to. And it wasn't just the free [Raspberry Pi][] or the
Wreck-it-Ralph swag from Disney or the fact that I stood next to [Guido][]
for a minute during the poster session. No, PyCon is just good people. The
Python community is diverse and accepting, and I can't list all the awesome,
kind people I met there.

There were, unfortunately, [disappointments][ctb-blog], but what other
tech conference has a sold-out full-day [education summit][], or
[raises $10k][pyladies-auction] for a community group, or [raises $6k][5k]
for cancer research and the [John Hunter Memorial fund][jhunter] with a
5k fun run? And PyCon attendees were 20% women! It's amazing to have been a
part of conference where community, generosity, and outreach were put front
and center. I tried to do my small part by giving people directions during the
tutorials.

Anyway, on to the specifics of what I did:

# Tutorials

The first tutorial I went to was called ["A beginner's introduction to Pydata:
how to build a minimal recommendation engine"][pydata-tut]. The intent of the
tutorial was to introduce [NumPy][] and [pandas][]. I was hoping to learn
some pandas-fu but I found the material poorly organized and didn't feel like
I was getting a good idea of why/when to use particular pandas features. The
video for this one doesn't seem to be up yet.

The second tutorial I went to was called ["Bayesian statistics made simple"]
[bayes-tut] and this one was awesome! I was comfortable with Bayesian stats
beforehand but a refresher never hurts and the instructor ([Allen Downey][])
gave terrific explanations. He had a little Bayesian stats library for us to
use in the programmatic examples, which was fun. (Though I had to re-compile
NumPy and SciPy to get it to work. It used the one little corner of SciPy
that's often broken on Macs.) If you're interested in learning more Downey is
working on a new book called [Think Bayes][] that you can read for free,
[Fernando Perez][fperez] has [posted his notebook][fperez-bayes] from the
course, and you can [watch the video][bayes-video].

# Education Summit

The [PyCon Education Summit][education summit] brought together educators from
all kinds of backgrounds from K-12 teachers to those teaching adults. I went
due to my interest as an instructor for [Software Carpentry][]. Most of the
discussion focused on teaching Python/computation in long-form courses to
people who have zero programming experience.

I didn't take much concrete away from the summit, but I was impressed with the
sheer level of energy going into the Python/education nexus. There are many
people out there experimenting with Python in education and developing
lessons that use Python. There are also a lot of user groups around the country
(like the [Boston Python Workshop][]) that are actively working to bring new
people into the Python world. Many people do this in their spare time! That's
the kind of community devotion I love about Python.

I gave a five minute lightning talk at the summit that was part a preview of my
PyCon talk and part showing off [ipythonblocks][]. The Notebooks for that are
at [nbviewer.ipython.org/5165758][edu-lightning].

# Talks

The first and most important thing you should know about the talks is that they
were all recorded and the [videos are online][pycon-talks]. There were about a
million concurrent talk sessions and I'm still catching up on all the great
stuff I missed. I highly recommend starting with the opening/closing statements
from [Jesse Noller][] and the Raspberry Pi keynote from Eben Upton:

- [Opening Address][opening]
- [Closing Address][closing]
- [Raspberry Pi][pi-note]

I think there were standing ovations during each of those. And then there were
the great regular talks I saw in person:

- [Python: A Toy Language][beazley] by Dave Beazley
    - Do not miss a chance to see Dave Beazley talk. You will be thoroughly
        entertained and leave wondering why you do such boring things
        with your code. Here Beazley talks about using Python to control a
        hobby CNC mill.
- [How the Internet works][mckellar] by Jessica McKellar
    - Learn about the underlying structure and protocol of the web!
- [Awesome Big Data Algorithms][titus] by Titus Brown
    - Titus gives a great introduction to some algorithms and data structures
        that help deal with Data of Unusual Size. Also check out his
        [blog post][titus-pycon-blog] on the talk with links to his notebooks.
- [Who's there? - Home Automation with Arduino/Raspberry Pi][rupa]
    by Rupa Dachere
    - Rupa tells us how she built an automated front door camera. This talk
        was standing room only!
- [What Teachers Really Need from Us][selena] by Selena Deckelmann
    - Selena relates her experience getting to know teachers and how we as
        developers can best help them.

## My Talk

I gave a talk titled ["Teaching with the IPython Notebook"][my-talk-listing]
that focused on how the [IPython Notebook][] can help students learning Python.
(Primarily by simplifying their interface to Python.) It seemed to go well
and I'm really glad I did it! The [video is up][my-talk] and my presentation
notebook is at [nbviewer.ipython.org/5165431][my-notebook].

# Posters

I stopped by [Simeon Franklin's poster][simeon-poster] about making Python more
beginner friendly and I was really impressed with the level of interest
surrounding the topic. Even Guido was there seriously engaged in this
dicussion. With engagement of this magnitude at that level I think we'll see
people putting serious effort into making Python more user friendly right
out of the box, which will be wonderful.

# Observations

As Wes McKinney [noted on Twitter][wes-tweet], there were two things everywhere
at PyCon this year: the IPython Notebook and Raspberry Pis. It seemed like
every other talk and tutorial was using the Notebook and it's no surprise, the
Notebook is so fantastic for presenting code plus supporting material and then
sharing it. It's a major boon to Python.

Everyone at the conference (plus some kids who came for [free tutorials]
[kids-python]) left with a [Raspberry Pi][]. These amazing little computers
enable all kinds of projects, often attached to an [Arduino][] for talking to
hardware. In Eben Upton's keynote I learned that the "Pi" in "Raspberry Pi" is
for Python since much of the system is built on Python. The site
[raspberry.io][] has been set up as a community of projects that use RPis but
I'm sure Googling turns up a ton more. A small, cheap, low powered, easy to
program computer just has so many possibilities! I haven't had a chance to
start hacking on mine yet but I'm looking forward to it!

# Thanks

A big thanks to [STScI][] for sending me. Thanks to [Greg Wilson][] for
suggesting the talk idea and thanks to [Titus Brown][] and [Ethan White][]
for looking over my proposal.

[PyCon 2013]: https://us.pycon.org/2013/
[Raspberry Pi]: http://www.raspberrypi.org/
[Guido]: https://twitter.com/gvanrossum
[ctb-blog]: http://ivory.idyll.org/blog/pycon-2013-and-codes-of-conduct.html
[education summit]: https://us.pycon.org/2013/events/edusummit/
[pyladies-auction]: http://www.marketwire.com/press-release/-1771597.htm
[5k]: https://twitter.com/jessenoller/statuses/315157894611992578
[jhunter]: http://numfocus.org/johnhunter/
[pydata-tut]: https://us.pycon.org/2013/schedule/presentation/28/
[NumPy]: http://www.numpy.org/
[pandas]: http://pandas.pydata.org
[bayes-tut]: https://us.pycon.org/2013/schedule/presentation/21/
[Allen Downey]: http://allendowney.com/
[Think Bayes]: http://www.greenteapress.com/thinkbayes/
[fperez-bayes]: http://nbviewer.ipython.org/urls/raw.github.com/fperez/bayestut/master/Bayes-1.ipynb
[fperez]: https://twitter.com/fperez_org
[bayes-video]: http://pyvideo.org/video/1724/bayesian-statistics-made-simple
[Software Carpentry]: http://software-carpentry.org
[Boston Python Workshop]: http://bostonpythonworkshop.com/
[ipythonblocks]: https://github.com/jiffyclub/ipythonblocks
[edu-lightning]: http://j.mp/jiffyclub-pycon-edu-2013
[pycon-talks]: http://pyvideo.org/category/33/pycon-us-2013
[Jesse Noller]: https://twitter.com/jessenoller
[opening]: http://pyvideo.org/video/1848/opening-statements
[closing]: http://pyvideo.org/video/1763/closing-address
[pi-note]: http://pyvideo.org/video/1668/keynote-2
[beazley]: http://pyvideo.org/video/1729/python-a-toy-language
[mckellar]: http://pyvideo.org/video/1677/how-the-internet-works
[titus]: http://pyvideo.org/video/1781/awesome-big-data-algorithms
[titus-pycon-blog]: http://ivory.idyll.org/blog/2013-pycon-awesome-big-data-algorithms-talk.html
[rupa]: http://pyvideo.org/video/1688/whos-there-home-automation-with-arduinorasp
[selena]: http://pyvideo.org/video/1697/what-teachers-really-need-from-us
[my-talk-listing]: https://us.pycon.org/2013/schedule/presentation/122/
[my-talk]: http://pyvideo.org/video/1744/teaching-with-the-ipython-notebook
[my-notebook]: http://j.mp/jiffyclub-pycon-talk-2013
[IPython Notebook]: http://ipython.org/notebook.html
[wes-tweet]: https://twitter.com/wesmckinn/status/312965439607169024
[kids-python]: https://us.pycon.org/2013/events/letslearnpython/
[Arduino]: http://www.arduino.cc/
[raspberry.io]: http://raspberry.io/
[simeon-poster]: http://simeonfranklin.com/blog/2013/mar/17/my-pycon-2013-poster/
[STScI]: http://www.stsci.edu/
[Greg Wilson]: http://third-bit.com
[Titus Brown]: http://ivory.idyll.org/blog/
[Ethan White]: http://jabberwocky.weecology.org/
