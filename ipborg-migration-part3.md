This is Part 3 in a series of blog posts describing my move of
[ipythonblogs.org][ipborg] from Rackspace to Heroku.
In this post I'll describe the updates I've made to the database interface
module of ipythonblocks.org.
Other posts are:

- [Part 1][]: Introduction and Architecture
- [Part 2][]: Data Migration
- Part 3: Database Interface Updates

The big change to the database interface module was the switch from [dataset][]
to [SQLAlchemy][] for database abstraction.
This involves using the ORM models described in [Part 2][],
removing the JSON de/serialization functions needed to use SQLite,
removing use of [memcached][],
and updating tests to use a Postgres database to match production.
The full diff is [here][dbi_diff], but I'll breakdown the important points below.

## SQLAlchemy ORM

With the [SQLAlchemy ORM][orm] you write Python classes representing your
tables and use instances of those classes to represent rows of data.
Here's how I'm now adding a new grid to the ipythonblocks.org database
using the [models][models.py] introduced in [Part 2][]:

```python
def store_grid_entry(session, grid_spec):
    """
    Add a grid spec to the database and return the grid's unique ID.

    Parameters
    ----------
    session : sqlalchemy.orm.session.Session
    grid_spec : dict

    Returns
    -------
    hash_id : str

    """
    table = models.SecretGrid if grid_spec['secret'] else models.PublicGrid
    new_grid = table(**grid_spec)
    session.add(new_grid)
    session.flush()

    return encode_grid_id(new_grid.id, grid_spec['secret'])
```

`session` is a [SQLAlchemy Session][sa-session] instance and is the primary
means of communicating with the database when using the ORM.
I'll talk more about where this comes from in my next post.
Note that after calling `session.flush()` above (which sends the new row to the
database) I can access `new_grid.id` on the same
instance I started with and SQLAlchemy will automatically load the primary key
ID assigned by the database when the row was added.
Reading a specific grid from the database is similarly brief:

```python
def get_grid_entry(session, hash_id, secret=False):
    """
    Get a specific grid entry.

    Parameters
    ----------
    session : sqlalchemy.orm.session.Session
    hash_id : str
    secret : bool, optional
        Whether this is a secret grid.

    Returns
    -------
    grid_spec : dict
        Will be None if no matching grid was found.

    """
    grid_id = decode_hash_id(hash_id, secret)
    if not grid_id:
        # couldn't do the conversion from hash to database ID
        return

    table = models.SecretGrid if secret else models.PublicGrid
    grid_spec = session.query(table).filter(table.id == grid_id).one_or_none()

    return grid_spec
```

The ORM has many features I'm not using here, for example taking advantage of
foreign keys to [automatically load linked entities][sa-relation].

## JSON

Some of the data I store for ipythonblocks.org can be represented by JSON,
i.e. lists and dictionaries containing strings and numbers (and other lists and dictionaries).
To store these data in [SQLite][sqlite] I used Python's [json][pyjson]
module to convert Python objects to strings to be stored as text in the database
(and to convert back to Python containers when reading).
With [Postgres' JSON types][pg-json], and [SQLAlchemy's support][sa-jsonb]
thereof, I don't have to think about this.
When I load records from the database, the JSON fields are converted straight
to Python containers:

```python
record = session.query(models.PublicGrid).filter(
    models.PublicGrid.id == grid_id).one()
record.grid_data  # this will be a dictionary containing lists, numbers, etc.
```

## Testing

#### Database Fixtures

For robust testing I want to test against a live database, not use mocks.
With SQLite this was relatively easy because a SQLite database is a single file,
but a Postgres database requires a running server.
Using [pytest][] [fixtures][] and the [testing.postgresql][] library it's
possible to temporarily create a Postgres database for the duration of tests.
This fixture uses `testing.postgresql` to create a database and a SQLAlchemy
engine to be shared by all the tests:

```python
@pytest.fixture(scope='module')
def pg_engine():
    with testing.postgresql.Postgresql() as postgresql:  # create database
        engine = sa.create_engine(postgresql.url())
        models.Base.metadata.create_all(bind=engine)  # create tables before tests

        yield engine

        engine.dispose()  # close any open connections
```

This fixture operates kind of like a context manager: the stuff before the
`yield engine` line happens as part of fixture setup before tests and
the stuff after happens after tests have run.
The `scope='module'` declares that this fixture needs to be evaluated only
once for all the tests in the module.

This next fixture creates a new connection, starts a transaction, and makes
a session for an individual test:

```python
@pytest.fixture
def session(pg_engine):
    conn = pg_engine.connect()  # create new connection to DB
    transaction = conn.begin()  # start a new transaction

    Session = sessionmaker(bind=conn)
    session = Session()  # create new session for tests

    yield session

    # rollback session/transaction to leave DB in a clean state
    # plus close resources
    session.rollback()
    session.close()
    Session.close_all()
    transaction.rollback()
    conn.close()
```

The setup portion of this fixture will be run before every test that uses
it and the teardown will be run after.
A critical piece of this fixture is that it starts and rolls back a transaction
in order to leave the database in a clean state for the next test.
If this fixture left the database in a dirty state the `pg_engine` fixture
would have create a fresh slate for tests by creating a brand new databse,
which is likely to be slower than issuing a transaction rollback.

#### Tests

With the database and connections taken care of it's now possible to test
the grid storage and retrieval code:

```python
@pytest.mark.parametrize('secret', [False, True])
def test_get_store_grid_entry(secret, basic_grid, session):
    data = basic_grid._construct_post_request(None, secret)

    # hack to normalize tuples to lists in the data dict so it matches JSON
    comp_data = json.loads(json.dumps(data))

    hash_id = dbi.store_grid_entry(session, data)

    grid_id = dbi.decode_hash_id(hash_id, secret)
    assert grid_id == 1

    grid_inst = dbi.get_grid_entry(session, hash_id, secret=secret)
    assert grid_inst.id == 1
    for key, value in comp_data.items():
        assert getattr(grid_inst, key) == value
```

The [`_construct_post_request`][post-req] method from ipythonblocks returns the
dictionary of grid data sent to ipythonblocks.org.
The test calls the storage and retrieval code from the database interface module
and compares the results to the original dictionary of data.

This test requires three inputs: `secret`, `basic_grid`, and `session`.
What are those and where do they come from?
`session` we've already seen from fixtures described above.
`basic_grid` is another fixture that provides an ipythonblocks `BlockGrid`
instance to generate test data.
Finally, this test uses [pytest's parametrization][pytest-param] feature to
run the same test code for multiple inputs, in this case to run the test
with both `secret=True` and `secret=False`.
Part of the magic of pytest is that it allows you to combine fixtures and
parametrization in this natural way.

## What's Next

In the next post I'll describe the updates to the Tornado application code,
in particular the changes required to provide SQLAlchemy sessions to the
database interface code when serving requests.

[ipborg]: http://ipythonblocks.org
[Part 1]: https://penandpants.com/2017/07/02/ipythonblocks-org-move-part-1/
[Part 2]: https://penandpants.com/2017/07/03/ipythonblocks-org-move-part-2-data-migration/
[dataset]: https://dataset.readthedocs.io/en/latest/
[SQLAlchemy]: http://www.sqlalchemy.org/
[memcached]: https://memcached.org/
[lru_cache]: https://docs.python.org/3.6/library/functools.html#functools.lru_cache
[dbi_diff]: https://github.com/jiffyclub/ipythonblocks.org/commit/d7412b4c3ee7e81a0a20662356f2a48ad0c7fea3
[orm]: http://docs.sqlalchemy.org/en/rel_1_1/orm/tutorial.html
[models.py]: https://github.com/jiffyclub/ipythonblocks.org/blob/d7412b4c3ee7e81a0a20662356f2a48ad0c7fea3/app/models.py
[sa-session]: http://docs.sqlalchemy.org/en/rel_1_1/orm/session.html
[sa-relation]: http://docs.sqlalchemy.org/en/rel_1_1/orm/tutorial.html#building-a-relationship
[sqlite]: https://docs.python.org/3/library/sqlite3.html
[Postgres]: https://www.postgresql.org/
[pyjson]: https://docs.python.org/3.6/library/json.html
[pg-json]: https://www.postgresql.org/docs/current/static/datatype-json.html
[sa-jsonb]: http://docs.sqlalchemy.org/en/rel_1_1/dialects/postgresql.html#sqlalchemy.dialects.postgresql.JSONB
[pytest]: https://docs.pytest.org/en/latest/
[fixtures]: https://docs.pytest.org/en/latest/fixture.html
[testing.postgresql]: https://github.com/tk0miya/testing.postgresql
[post-req]: https://github.com/jiffyclub/ipythonblocks/blob/ba6311a24ae31fcfe7f25b769dda6002303e7835/ipythonblocks/ipythonblocks.py#L767
[pytest-param]: https://docs.pytest.org/en/latest/parametrize.html#pytest-mark-parametrize-parametrizing-test-functions
