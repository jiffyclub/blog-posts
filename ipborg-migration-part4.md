This is Part 4 in a series of blog posts describing my move of
[ipythonblogs.org][ipborg] from Rackspace to Heroku.
In this post I'll describe the updates I've made to the application layer
of ipythonblocks.org.
Other posts are:

- [Part 1][]: Introduction and Architecture
- [Part 2][]: Data Migration
- [Part 3][]: Database Interface Updates
- Part 4: Application Updates

The application logic is not really changed in this update,
the bulk of changes are to support providing [SQLAlchemy][] sessions
to allow database access during requests.
(See [Part 3][] for discussion of the database interface layer of ipythonblocks.org.)

## Application Overview

ipythonblocks.org is powered by [Tornado][],
which combines an application framework, web server, and asynchronous features.
On Heroku the application is started with the command

```shell
python -m app.app --port=$PORT --db_url=$DATABASE_URL
```

`$PORT` and `$DATABASE_URL` are environment variables provided by Heroku
to specify, respectively, which port to listen on and where to find the
Postgres database attached to the app.
These are parsed from the command line by [Tornado's options module][to-options]
and made available globally on the `tornado.options.options` variable.

## SQLAlchemy Sessions

#### Engine and Session Factory

ipythonblocks.org uses the the [SQLAlchemy ORM][orm] to interact with the
database, which requires [sessions][sa-session].
I could in theory create new connections to the database inside of every
function in the database interface, but it's more efficient to let SQLAlchemy
manage connections via an [Engine][sa-engine] and [session factory][sa=factory].
These are meant to live for the lifetime of an application process,
so they can be created at application startup time.
With [Tornado][] this can be accomplished by creating a subclass of the
[Tornado application class][to-app] and adding engine/sessionmaker
creation to its `__init__`:

```python
class AppWithSession(tornado.web.Application):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.engine = sa.create_engine(tornado.options.options.db_url)
        self.session_factory = sessionmaker(bind=self.engine)
```

An application instance is created once at application startup time
and made available to request handlers.
The `tornado.options.options.db_url` variable is the result of the command-line
option parsing mentioned above.

#### Request Scoped Sessions

The engine and session factory help manage database connections,
but for the actual database communication I need [session][sa-session] instances.
The goal is to have one session for every request that is opened at the
beginning of the request and closed at the end.
Tornado has [`prepare`][to-prepare] and [`on_finish`][to-finish] methods
that are run before and after requests, but the downside of these is that
they are disconnected from the request logic and don't give you the opportunity
to catch exceptions and rollback changes.
I decided to add a context manager to my request handlers instead and to
wrap all DB accesses within the context manager.
Here's how the context manager is added to a specialized subclass of
Tornado's [request handler class][to-request]
(this context manager is borrowed straight [from the SQLAlchemy docs][sa-context]):

```python
class DBAccessHandler(tornado.web.RequestHandler):
    @contextlib.contextmanager
    def session_context(self):
        session = self.application.session_factory()
        try:
            yield session
            session.commit()
        except:
            session.rollback()
            raise
        finally:
            session.close()
```

This can then be called from within request handlers to get a session
without having to worry about committing, rolling back, or closing it.
For example, here's the request handler for [ipythonblocks.org/random][random]:

```python
class RandomHandler(DBAccessHandler):
    def get(self):
        with self.session_context() as session:
            hash_id = dbi.get_random_hash_id(session)

        self.redirect('/' + hash_id, status=303)
```

## Testing the App

#### Fixtures

Testing the application requires a slightly different database strategy
than I used in [Part 3][] for the database interface module.
There nothing was ever committed to the database so at the end of each
test I could rollback an open transaction to restore the DB,
but while testing the application things will be committed.
To restore the database for the next test I'll have [testing.postgresql][]
fully teardown and rebuild the database.
Luckily testing.postgresql has a [feature][tpg-faster] that makes this process
slightly faster by caching the initial database files for re-use and
I use this feature when setting up a test database factory at the test module
level:

```python
def setup_module(module):
    module.PG_FACTORY = testing.postgresql.PostgresqlFactory(
        cache_initialized_db=True)

def teardown_module(module):
    module.PG_FACTORY.clear_cache()
```

`setup_module` and `teardown_module` are [features][pytest-xunit] of [pytest][]
that are run once before and after all the tests in the module.
The `module` argument passed in is the test module itself, which is a bit of a trip.
It can be used to set global values within the module namespace that can be
accessed later.
I use this tactic here instead of pytest's amazing [fixtures][] because
to test a Tornado app I have to use [unittest][] subclasses,
which are not compatible with fixtures. (More on that below.)

At the test level I use these test fixtures to create sessions and configure
the `tornado.options.options` variable with the URL of the test database:

```python
class UtilBase(tornado.testing.AsyncHTTPTestCase):
    def setup_method(self, method):
        self.postgresql = PG_FACTORY()
        self.engine = sa.create_engine(self.postgresql.url())
        models.Base.metadata.create_all(bind=self.engine)
        self.Session = sessionmaker(bind=self.engine)
        self.session = self.Session()
        tornado.options.options.db_url = self.postgresql.url()

    def teardown_method(self, method):
        self.session.close()
        self.Session.close_all()
        self.engine.dispose()
        self.postgresql.stop()
```

These are run before and after each test method in the file.

#### Testing Tornado Apps

Testing the application means simulating real requests that return
synchronously within a test.
That would be complicated if I had to manage it on my own, but most
web frameworks/servers come with utilities to help manage testing,
including Tornado.
In the case of Tornado I need to subclass [`AsyncHTTPTestCase`][to-testing],
which is itself a subclass of [`unittest.TestCase`][TestCase].
This means I can't take advantage of many of my favorite pytest fixtures,
but I want to take advantage of Tornado's testing features.
You can see above that I've already subclassed `AsyncHTTPTestCase` to
create a test superclass with fixtures, so my test cases subclass `UtilBase`.
As an example, here's a couple of the tests of the `/post` endpoint for
sending new grids to ipythonblocks.org:

```python
class TestPostGrid(UtilBase):
    app_url = '/post'
    method = 'POST'

    def test_json_failure(self):
        response = self.get_response('{"asdf"}')
        assert response.code == 400

    def test_validation_failure(self):
        response = self.get_response('{"asdf": 5}')
        assert response.code == 400

    def test_returns_url(self):
        req = request()
        response = self.get_response(json.dumps(req))

        assert response.code == 200
        assert 'application/json' in response.headers['Content-Type']

        body = json.loads(response.body)
        assert body['url'] == 'http://www.ipythonblocks.org/bizkiL'
```

To see more, the bulk of the application code changes are in
[this commit][app-changes].

## What's Next

That's it! There were other changes that I haven't highlighted in these
posts, mostly related to logging now that I'm logging only to
[standard out][stdout] and not to files.
If you're interested in the full set of differences made while doing this
migration from Rackspace to Heroku you can see them [here][full-diff].

ipythonblocks.org is now up and running on Heroku: http://www.ipythonblocks.org/
(The URL now contains a `www.` because of [Heroku's architecture][custom-domains]
and Hover [not supporting][hover-no] compatible [DNS][] records.)
ipythonblocks has the updated URL as of version 1.7.1 and I've tested it
with a [new grid][new-grid].
Please give it a try and thanks for reading!

[ipborg]: http://www.ipythonblocks.org
[Part 1]: https://penandpants.com/2017/07/02/ipythonblocks-org-move-part-1/
[Part 2]: https://penandpants.com/2017/07/03/ipythonblocks-org-move-part-2-data-migration/
[Part 3]: https://penandpants.com/2017/07/10/ipythonblocks-org-move-part-3-database-interface/
[SQLAlchemy]: http://www.sqlalchemy.org/
[orm]: http://docs.sqlalchemy.org/en/rel_1_1/orm/tutorial.html
[sa-session]: http://docs.sqlalchemy.org/en/rel_1_1/orm/session.html
[sa-engine]: http://docs.sqlalchemy.org/en/rel_1_1/core/connections.html#basic-usage
[sa-factory]: http://docs.sqlalchemy.org/en/rel_1_1/orm/session_api.html#sqlalchemy.orm.session.sessionmaker
[sa-context]: http://docs.sqlalchemy.org/en/rel_1_1/orm/session_basics.html#when-do-i-construct-a-session-when-do-i-commit-it-and-when-do-i-close-it
[Tornado]: http://www.tornadoweb.org/en/stable/
[to-options]: http://www.tornadoweb.org/en/stable/options.html
[to-prepare]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.prepare
[to-finish]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.RequestHandler.on_finish
[to-request]: http://www.tornadoweb.org/en/stable/web.html#request-handlers
[to-app]: http://www.tornadoweb.org/en/stable/web.html#application-configuration
[to-testing]: http://www.tornadoweb.org/en/stable/testing.html#tornado.testing.AsyncHTTPTestCase
[random]: http://www.ipythonblocks.org/random
[testing.postgresql]: https://github.com/tk0miya/testing.postgresql
[tpg-faster]: https://github.com/tk0miya/testing.postgresql#to-make-your-tests-faster
[pytest-xunit]: https://docs.pytest.org/en/latest/xunit_setup.html
[pytest]: https://docs.pytest.org/en/latest/
[fixtures]: https://docs.pytest.org/en/latest/fixture.html
[unittest]: https://docs.python.org/3.6/library/unittest.html
[TestCase]: https://docs.python.org/3.6/library/unittest.html#unittest.TestCase
[app-changes]: https://github.com/jiffyclub/ipythonblocks.org/commit/ca35047922cdc01546f52f671a99d25c1598ef4d
[stdout]: https://en.wikipedia.org/wiki/Standard_streams#Standard_output_.28stdout.29
[full-diff]: https://github.com/jiffyclub/ipythonblocks.org/compare/4dd6289...b64efa9
[custom-domains]: https://devcenter.heroku.com/articles/custom-domains
[hover-no]: https://help.hover.com/hc/en-us/community/posts/209217687-ALIAS-or-ANAME-records
[DNS]: https://en.wikipedia.org/wiki/Domain_Name_System
[new-grid]: http://www.ipythonblocks.org/bTxGkc
