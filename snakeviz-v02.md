It has been over two years since [Erik Bray][embray] and I made the
first release of [SnakeViz 0.1][SnakeViz01].
It had multiple performance bottlenecks,
but it worked just well enough that it took me a long time to
prioritize making improvements.
That time has finally come around and I'm happy to announce that
SnakeViz 0.2 is [now available][pypi]!

# What's New

The look and feel of SnakeViz remains much the same (see a [screenshot][]),
but there are some new things on the screen:

![SnakeViz screenshot][screenshot]

- Detailed function information when hovering over the visualization
- Call stack list for tracking where you are when zooming the visualization
- Control the depth of the displayed call tree
- Limit the display of functions that take up relatively little time

## Under the Hood

The first release of SnakeViz had some performance bottlenecks:

- It tried to transfer a complete call tree from the server to the client
    as JSON
- It tried to display the entire call tree in the sunburst visualization

Those limited the usefulness of SnakeViz with profiles that contained calls
to a lot of functions.
The version 0.2 release is an almost complete rewrite in order to make
SnakeViz work with larger profiles.

The first limitation is addressed by moving the building of call trees
into the client application.
Profile data is passed from the server to the client in close to the
same form as it's available from Python's [pstats][] module.
Once in the client, the profile data is used to construct call trees
on demand for visualization.

The second limitation is addressed by limiting how much of the
profile is visualized at once.
Call trees are built only to a user specified depth and users
can opt to omit functions that do not use much time from display.
(The "depth" and "cutoff" controls.)

----

I and others have tested SnakeViz 0.2 with some fairly large
profiles and found it works.
You can read more about SnakeViz in the [updated docs][docs].
Please give it a try!
Issues can be reported [on GitHub][gh].

[embray]: http://iguananaut.net/
[SnakeViz01]: http://penandpants.com/2012/09/21/snakeviz-0-1-a-python-profile-viewer/
[pypi]: https://pypi.python.org/pypi/snakeviz
[screenshot]: https://raw.githubusercontent.com/jiffyclub/blog-posts/master/images/snakeviz-v02/snakeviz_screenshot.png
[pstats]: https://docs.python.org/2/library/profile.html#module-pstats
[docs]: https://jiffyclub.github.io/snakeviz/
[gh]: https://github.com/jiffyclub/snakeviz
