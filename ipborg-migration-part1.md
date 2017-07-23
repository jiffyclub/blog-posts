This is Part 1 in a series of blog posts describing my move of
[ipythonblogs.org][ipborg] from Rackspace to Heroku.
In this first post I'll describe the existing deployment and what I intend
to change during the migration.
Other posts are:

- Part 1: Introduction and Architecture
- [Part 2][]: Data Migration
- [Part 3][]: Database Interface
- [Part 4][]: Application Updates

As a side project I maintain a Python library called [ipythonblocks][]
that displays colored grids in a Jupyter Notebook.
(See also [this intro blog post][ipb-intro].)
It can be useful for teaching or for a bit of fun art.
I also maintain the website [ipythonblocks.org][ipborg]
that allows users to post their ipythonblocks grids on the internet to be shared.
ipythonblocks.org has been hosted on [Rackspace][] since I [first launched it][ipborg-announce],
but now I'm migrating the site to [Heroku][] for easier maintenance
and deployment.

## Rackspace Deployment

ipythonblocks.org has been running on a single Rackspace computer with 2 GB of RAM.
I deployed it by copying code up over SSH then connecting over SSH and
running a shell script. Very low tech!
There were two [Docker][] containers running, one for the Python app that
served the site and one for [memcached][] (used to cache the results of
database lookups). The grid data sent to ipythonblocks.org was stored in a
[sqlite][] database on disk.

The application itself uses [Tornado][] to build and serve HTML pages,
there's no client side JavaScript.
You can read more about the libraries used in the app in
[this previous blog post][ipborg-libraries].
You can browse the application code on GitHub in its state prior to the migration
[here](https://github.com/jiffyclub/ipythonblocks.org/tree/4dd628924d78e4cb6dc1053645fc263b1d517465).

## Why Change Anything?

The site periodically goes down for unknown reasons and I'm over SSH-ing into
the computer to restart things until it works again.
I've been hosting [Days Since it Rained][dsir] on Heroku for years and
never had to think about it and I'd like that same level of reliability
for ipythonblocks.org.
The site gets practically no traffic so I can easily use Heroku's free tier.

## Planned Changes

The biggest change will be not using Docker anymore when Heroku is handling
all the server stuff for me.
I'll be able to stop using all code related to deployment and focus only
on application code. I'm also planning to make some other changes:

#### Update to Python 3.6

I mean, of course.

#### SQLite to Postgres

Heroku doesn't have a persistent filesystem so I can't use SQLite,
but Heroku does offer a [Postgres][] add-on with a free tier.
As an added bonus I can use Postgres' [JSON types][pg-json],
which are a good fit for some of the data I store.
(To store JSON-like data in SQLite I wrote some wrapper code to
convert it to/from strings.)

#### Dataset to SQLAlchemy

I originally used [dataset][] as a SQL abstraction for ipythonblocks.org.
I'm switching to [SQLAlchemy][] so I can use Postgres' JSON data type
and so I can use SQLAlchemy's ORM, which is a good fit for a web app.

#### No memcached

Caching is a really great idea in web apps, but ipythonblocks.org sees so
little traffic that I think I'll be able to read from the DB for every request
without problem and simplify the app logic.

## What's Next

In my next post I'll describe the grid data model and how I migrated the
existing grid data from SQLite to Postgres.

[Part 2]: https://penandpants.com/2017/07/03/ipythonblocks-org-move-part-2-data-migration/
[Part 3]: https://penandpants.com/2017/07/10/ipythonblocks-org-move-part-3-database-interface/
[Part 4]: https://penandpants.com/2017/07/22/ipythonblocks-org-move-part-4-application-updates/
[ipythonblocks]: https://github.com/jiffyclub/ipythonblocks
[ipb-intro]: https://penandpants.com/2013/01/10/ipythonblocks-a-visual-tool-for-practicing-python/
[ipborg]: http://ipythonblocks.org
[ipborg-announce]: https://penandpants.com/2014/02/14/announcing-ipythonblocks-org/
[Rackspace]: https://www.rackspace.com/
[Heroku]: https://www.heroku.com/
[ipborg-libraries]: https://penandpants.com/2014/02/20/the-libraries-of-ipythonblocks-org/
[Docker]: https://www.docker.com/
[memcached]: https://memcached.org/
[sqlite]: https://docs.python.org/3/library/sqlite3.html
[Tornado]: http://www.tornadoweb.org/en/stable/
[dsir]: http://www.dayssinceitrained.com/
[Postgres]: https://www.postgresql.org/
[pg-json]: https://www.postgresql.org/docs/current/static/datatype-json.html
[dataset]: https://dataset.readthedocs.io/en/latest/
[SQLAlchemy]: http://www.sqlalchemy.org/
