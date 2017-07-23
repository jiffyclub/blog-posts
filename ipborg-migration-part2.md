This is Part 2 in a series of blog posts describing my move of
[ipythonblogs.org][ipborg] from Rackspace to Heroku.
In [Part 1][] of this series I described my motivation for the move
and the broad changes I expect to make as part of the migration.
In this post I'll describe the grid data model and how I migrated the
existing grid data from [SQLite][] to [Postgres][].
Other posts are:

- [Part 1][]: Introduction and Architecture
- Part 2: Data Migration
- [Part 3][]: Database Interface Updates
- [Part 4][]: Application Updates

## SQLite Data Model

Grids are stored in a table with this schema:

```sql
sqlite> .schema --indent public_grids
CREATE TABLE public_grids(
  id INTEGER NOT NULL,
  ipb_version TEXT,
  python_version TEXT,
  grid_data TEXT,
  secret BOOLEAN,
  code_cells TEXT,
  ipb_class TEXT,
  PRIMARY KEY(id)
);
```

Simple enough, but the `python_version`, `grid_data`, and `code_cells`
columns are actually encoded JSON data.
In the application code all reads and writes are wrapped by code that
calls `json.loads` or `json.reads`.
Still, this a relatively simple data model compared with most web applications
where there are many tables linked by foreign keys and other relationships.
Here there's one table that holds everything.
This would have been a good fit for a document database like [MongoDB][],
but it also works fine in SQL.

(The DB actually contains two tables with identical schemas, one for public grids
and one for secret grids.
The main practical difference between public and secret grids is that
secret grids can't be discovered via the "random grid" feature on ipythonblocks.org.)

## Postgres Data Model

Postgres supports [JSON data types][pg-json] so I'll be able to dispense
with the manual de/serialization code.
I'm also using [SQLAlchemy][] now so the tables are defined in Python:

```python
"""Definition of SQL tables used to store ipythonblocks grid data"""
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql as pg
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()


class CommonColumnsMixin:
    """Columns common to both public and secret grids"""
    __table_args__ = {'schema': 'public'}  # "public" is postgres' default schema

    id = sa.Column(sa.Integer, primary_key=True)
    ipb_version = sa.Column(sa.Text, nullable=False)
    python_version = sa.Column(pg.JSONB, nullable=False)
    grid_data = sa.Column(pg.JSONB, nullable=False)
    code_cells = sa.Column(pg.JSONB)
    ipb_class = sa.Column(sa.Text, nullable=False)
    created_at = sa.Column(
        sa.DateTime(timezone=True), nullable=False,
        server_default=sa.text('NOW()'))


class PublicGrid(CommonColumnsMixin, Base):
    """Table to hold public grids (discoverable via "random")"""
    __tablename__ = 'public_grids'

    # no-op column, but put it here anyway
    secret = sa.Column(sa.Boolean, nullable=False, default=False)


class SecretGrid(CommonColumnsMixin, Base):
    """Table to hold secret grids"""
    __tablename__ = 'secret_grids'

    # no-op column, but put it here anyway
    secret = sa.Column(sa.Boolean, nullable=False, default=True)

```

This is using [SQLAlchemy's ORM][orm] and a [mixin class][mixins] to store
the common columns.

With SQLAlchemy's support for Postgres' JSON types the JSON data stored in the
tables will automatically be converted to Python lists and dictionaries
when I load query results (and it also works the other way when writing data).
(You can even [write queries][json-query] based on things inside of JSON columns,
but that's not a feature I'm using in the app.)

The one thing I've added to the data model is a `created_at` column that
will record when a grid was sent to ipythonblocks.org.
(`server_default=sa.text('NOW()')` means that the Postgres `NOW()` function
will be called to provide a value if one is not provided by the client during
an `INSERT` query.)

## Migration

With the tables defined I'm ready to copy data from SQLite to
Postgres.
I used `scp` to copy the SQLite database file from the ipythonblocks.org
server to my laptop so I could run the migration locally.
(Everything in a SQLite database is stored in a single file and
it's beautiful how the database file is platform-independent.)
Then I wrote a Python script to copy the data from SQLite to Postgres:

```python
"""Script for migrating ipythonblocks grid data from SQLite to Postgres"""
import contextlib
import json
import os
from pathlib import Path

import sqlalchemy as sa
from sqlalchemy.orm import sessionmaker

# The module in the ipythonblocks.org application code that contains
# table definitions
from app import models

# SQLite DB related variables
SQLITEDB = 'sqlite:///' + str(Path.home() / 'ipborg.db')
SQLITE_ENGINE = sa.create_engine(str(SQLITEDB))
SQLITE_META = sa.MetaData(bind=SQLITE_ENGINE)

# Postgres DB related variables
DBURL = os.environ['DATABASE_URL']  # could be local or remote server
PSQL_ENGINE = sa.create_engine(DBURL)
SESSION = sessionmaker(bind=PSQL_ENGINE)

# columns that are serialized JSON in the SQLite DB
JSONIZE_KEYS = {'python_version', 'code_cells', 'grid_data'}

# drop and recreate tables in the destination DB so we're always
# starting fresh
models.Base.metadata.drop_all(bind=PSQL_ENGINE)
models.Base.metadata.create_all(bind=PSQL_ENGINE)


def sqlite_row_to_sa_row(row, sa_cls):
    """
    Convert a row from the SQLite DB (a SQLAlchemy RowProxy instance)
    into an ORM instance such as PublicGrid or SecretGrid
    (exact class provided by the sa_cls argument). This takes care of
    de-serializing the JSON data stored in the SQLite DB.

    """
    d = dict(row)
    for key in JSONIZE_KEYS:
        d[key] = json.loads(d[key]) if d[key] else None

    return sa_cls(**d)


def sqlite_table_to_sa_rows(table_name, sa_cls):
    """
    Yields SQLAlchemy ORM instances of sa_cls from a SQLite table
    specified by table_name.

    """
    table = sa.Table(table_name, SQLITE_META, autoload=True)
    results = SQLITE_ENGINE.execute(table.select())
    for row in results:
        yield sqlite_row_to_sa_row(row, sa_cls)


@contextlib.contextmanager
def session_context():
    session = SESSION()
    try:
        yield session
        session.commit()
    except:
        session.rollback()
        raise
    finally:
        session.close()

def migrate():
    """
    Trigger the reading from SQLite, transformation of JSON data,
    and writing to Postgres.

    """
    with session_context() as session:
        session.add_all(sqlite_table_to_sa_rows('public_grids', models.PublicGrid))
        session.add_all(sqlite_table_to_sa_rows('secret_grids', models.SecretGrid))

        # Because all the grids added so far already had IDs, the sequences
        # backing the id columns in the grid tables haven't been advanced
        # at all. When trying to add a new table the sequence would provide
        # a key of 1, which would then collide with the existing grids.
        # We need to manually set the sequences behind the table primary keys
        # so that when new grids are added with no IDs the automatically
        # generated IDs are actually available.
        max_public_id = session.query(sa.func.max(models.PublicGrid.id)).scalar()
        max_secret_id = session.query(sa.func.max(models.SecretGrid.id)).scalar()

        session.execute(sa.text(
            f'select setval(\'public_grids_id_seq\', {max_public_id})'))
        session.execute(sa.text(
            f'select setval(\'secret_grids_id_seq\', {max_secret_id})'))


if __name__ == '__main__':
    migrate()

```

The total number of rows involved was less than 100 so I could easily
load all the data into memory at once and didn't worry about streaming.
I ran this to copy the data both to a local testing database and to the
production database on Heroku.

## What's Next

In the next post I'll describe updating the application database access
code to use SQLAlchemy and remove the use of memcached.

[ipborg]: http://ipythonblocks.org
[Part 1]: https://penandpants.com/2017/07/02/ipythonblocks-org-move-part-1/
[Part 3]: https://penandpants.com/2017/07/10/ipythonblocks-org-move-part-3-database-interface/
[Part 4]: https://penandpants.com/2017/07/22/ipythonblocks-org-move-part-4-application-updates/
[SQLite]: https://docs.python.org/3/library/sqlite3.html
[Postgres]: https://www.postgresql.org/
[MongoDB]: https://www.mongodb.com/
[pg-json]: https://www.postgresql.org/docs/current/static/datatype-json.html
[SQLAlchemy]: http://www.sqlalchemy.org/
[orm]: http://docs.sqlalchemy.org/en/rel_1_1/orm/tutorial.html
[mixins]: http://docs.sqlalchemy.org/en/rel_1_1/orm/extensions/declarative/mixins.html
[json-query]: https://www.postgresql.org/docs/current/static/functions-json.html
