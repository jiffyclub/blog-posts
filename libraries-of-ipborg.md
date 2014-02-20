In this post I'll describe the libraries used by [ipythonblocks.org][] to turn
requests into web pages and JSON to send back to users. In some future posts
I'll describe how it's actually put on the internet. If you're curious about
the code you can [see it on GitHub][ipborg-gh].

# Back End

The back end consists of `GET` and `POST` REST endpoints for [ipythonblocks][]
to talk to and handlers for the site itself: main and about pages, a random
grid redirect, and the individual grid views. In all there are about six
handlers for all of ipythonblocks.org.

#### Framework

ipythonblocks.org is such a simple site that any lightweight framework could
probably handle it. I went with [Tornado][] mainly because I've used it before
and I like the way applications are designed using Tornado. That it includes a
template engine and a high performance web server are also pluses. If I'd not
used Tornado, [Flask][] and [Jinja2][] would have been my second choice.

#### Database

Choosing a database was something of an agonizing decision.
You can choose from SQL, NoSQL, and key-value stores; and within each
of those you have many more choices. I like the simplicity of working with
schema-less databases like [MongoDB][], and I was very intrigued by
[RethinkDB][], but in the interest of having a simple setup that allowed
me to focus on developing app logic I ended up using [sqlite][].
I use the [dataset][] library to take care of some of the SQL overhead
(like table creation) so that I can combine the simplicity of sqlite with
a more NoSQL-like interface.

At some point I may want to move to another database, especially one running
on a dedicated machine so that swapping the application server can be done
without worrying about the database. When I get to that point I'll probably
take another look at [RethinkDB][] and see if it's ready for my application.

To avoid database lookups of recently visited pages I'm using [memcached][]
and talking to it from Python via the [pylibmc][] library.

#### Logging

Python's built in [logging][] can certainly get the job done, but its
interface has some rough edges I don't like. [Configuration][log-config]
can be painful for sophisticated cases and any kind of
[structured logging][] requires custom formatting. I think [Twiggy][] is a
much more "Pythonic" approach to logging with simpler configuration and
built in structured logging. ipythonblocks.org was my first time using
Twiggy and I'd use it again. (Though it is unfortunately not Python 3
compatible at this time.)

#### Other

Requests to the POST endpoint are validated using [jsonschema][].
This provides protection for the app against incorrectly configured requests
and can be used as a kind of documentation on what requests should look
like.

I use the [hashids][] library to turn the integer SQL IDs of grid entries
into short strings, as in http://ipythonblocks.org/zcezcM. This is a URL
form people are familiar with and it allows the implementation of "secret"
grid posts that have public URLs but are difficult to find unless someone
gives you the URL.

Users of ipythonblocks can include code with their posted grids and I use
[Pygments][] to highlight the syntax of the code and format it for HTML.
Pygments is decent enough to escape HTML included in the posted code so
I don't have to worry about that breaking the page rendering. The color
scheme used is Base16 Chalk Light via
https://github.com/idleberg/base16-pygments.

Finally, I use [ipythonblocks][] itself to turn grid data into rendered
HTML via the same methods used by the [IPython Notebook][].

# Front End

The back end renders and delivers static HTML to browsers (or JSON to
ipythonblocks) so there isn't much fancy going on in the front end.
I use [CSS media queries][] to adjust the site margins for small screens,
and on the [front page][ipythonblocks.org] I use [Pure CSS grids][] to
make a responsive three-column layout that collapses to a single column on
small screens.

ipythonblocks.org uses the [Source][] family of fonts from Adobe
delivered by [Google Fonts][].

[ipythonblocks.org]: http://ipythonblocks.org
[ipborg-gh]: https://github.com/jiffyclub/ipythonblocks.org
[ipythonblocks]: https://github.com/jiffyclub/ipythonblocks
[Tornado]: http://www.tornadoweb.org/
[Flask]: http://flask.pocoo.org/
[Jinja2]: http://jinja.pocoo.org/
[MongoDB]: http://www.mongodb.org/
[RethinkDB]: http://www.rethinkdb.com/
[sqlite]: http://docs.python.org/2/library/sqlite3.html
[dataset]: https://dataset.readthedocs.org
[memcached]: http://memcached.org/
[pylibmc]: http://sendapatch.se/projects/pylibmc/
[logging]: http://docs.python.org/2/library/logging.html
[log-config]: http://docs.python.org/2/howto/logging.html#configuring-logging
[structured logging]: http://docs.python.org/2/howto/logging-cookbook.html#implementing-structured-logging
[Twiggy]: https://twiggy.readthedocs.org
[jsonschema]: http://python-jsonschema.readthedocs.org/
[hashids]: http://www.hashids.org/python/
[Pygments]: http://pygments.org/
[IPython Notebook]: http://ipython.org/notebook.html
[CSS media queries]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries
[Pure CSS grids]: http://purecss.io/grids/
[Source]: https://www.google.com/fonts#ReviewPlace:refine/Collection:Source+Sans+Pro|Source+Code+Pro
[Google Fonts]: https://www.google.com/fonts
