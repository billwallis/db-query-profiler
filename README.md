<div align="center">

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![uv](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/uv/main/assets/badge/v0.json)](https://github.com/astral-sh/uv)
[![tests](https://github.com/billwallis/db-query-profiler/actions/workflows/tests.yaml/badge.svg)](https://github.com/billwallis/db-query-profiler/actions/workflows/tests.yaml)
[![coverage](https://raw.githubusercontent.com/billwallis/db-query-profiler/main/coverage.svg)](https://github.com/dbrgn/coverage-badge)
![GitHub last commit](https://img.shields.io/github/last-commit/billwallis/db-query-profiler)

[![code style: prettier](https://img.shields.io/badge/code_style-prettier-ff69b4.svg?style=flat-square)](https://github.com/prettier/prettier)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)
[![pre-commit.ci status](https://results.pre-commit.ci/badge/github/billwallis/db-query-profiler/main.svg)](https://results.pre-commit.ci/latest/github/billwallis/db-query-profiler/main)

</div>

---

# Database Query Profiler 🗃️⏱️

Lightweight database query profiler.

This tool is database-agnostic -- just provide a class that connects to your database with an `execute` method, and the queries that you want to profile.

> [!WARNING]
>
> **_This is NOT a replacement for analysing the [query plan](https://en.wikipedia.org/wiki/Query_plan). This should just support the analysis done with it._**

## Installation ⬇️

Grab a copy from PyPI like usual:

```
pip install db-query-profiler
```

If you'd prefer, you can install from source:

```
pip install git+https://github.com/billwallis/db-query-profiler.git@main
```

## Sample Output 📝

Given a set of queries (details below), this package prints the average time in seconds taken to run each query, as well as the percentage of the total time taken by each query.

The [`tqdm`](https://github.com/tqdm/tqdm) package is used to show progress of the queries being run.

A typical output will look something like this:

```
Start time: 2023-05-07 12:38:06.879738
----------------------------------------
100%|██████████| 5/5 [00:01<00:00,  3.29it/s]
query-1.sql: 0.10063192s (33.4%)
query-2.sql: 0.20044784s (66.6%)
----------------------------------------
End time: 2023-05-07 12:38:08.757555
```

## Usage 📖

The package exposes a single function, `time_queries`, which currently requires:

1. A database connection/cursor class that implements an `execute` method.
2. The number of times to re-run each query.
3. A directory containing the SQL files with the queries to run.

There should only be a single query in each file, and the file name will be used as the query name in the output.

For the following examples, assume that there are SQL files in the `queries` directory.

### SQLite Example

> Official documentation: https://docs.python.org/3/library/sqlite3.html

```python
import sqlite3

import db_query_profiler


def main() -> None:
    db_conn = sqlite3.connect(":memory:")  # Or a path to a database file
    db_query_profiler.time_queries(
        conn=db_conn,
        repeat=5,
        directory="queries"
    )


if __name__ == "__main__":
    main()
```

### Snowflake Example

> Official documentation: https://docs.snowflake.com/en/developer-guide/python-connector/python-connector-example

Some databases, like Snowflake, have [extra layers of caching](https://docs.snowflake.com/en/user-guide/querying-persisted-results) that can affect the results of the profiling. To avoid this and make the runtime comparisons more genuine, it's recommended to turn off these extra caching options (where this is supported).

```python
import db_query_profiler
import snowflake.connector  # snowflake-connector-python


# This dictionary is just for illustration purposes and
# you should use whatever connection method you prefer
CREDENTIALS = {
    "user": "XXX",
    "password": "XXX",
    "account": "XXX",
    "warehouse": "XXX",
    "role": "XXX",
    "database": "XXX",
}


def main() -> None:
    db_conn = snowflake.connector.SnowflakeConnection(**CREDENTIALS)
    with db_conn.cursor() as cursor:
        cursor.execute("""ALTER SESSION SET USE_CACHED_RESULT = FALSE;""")
        db_query_profiler.time_queries(
            conn=cursor,
            repeat=5,
            directory="queries",
        )
        cursor.execute("""ALTER SESSION SET USE_CACHED_RESULT = TRUE;""")
    db_conn.close()


if __name__ == "__main__":
    main()
```

## Warnings ⚠️

This package will open and run all the files in the specified directory, so be careful about what you put in there -- potentially unsafe SQL commands could be run.

This package only reads from the database, so it's encouraged to configure your database connection in a read-only way.

### SQLite

> Official documentation:
>
> - https://docs.python.org/3/library/sqlite3.html#sqlite3.connect
> - https://docs.python.org/3/library/sqlite3.html#how-to-work-with-sqlite-uris

To connect to a SQLite database in a read-only way, use the `uri=True` parameter with `file:` and `?mode=ro` surrounding the database path when connecting:

```python
db_conn = sqlite3.connect("file:path/to/database.db?mode=ro", uri=True)
```

## Contributing 🤝

The Python packaging is managed with [uv](https://github.com/astral-sh/uv), but that should be the only dependency.

To get started, just clone the repo, install the dependencies, and enable [pre-commit](https://pre-commit.com/):

```bash
uv sync --all-groups
pre-commit install --install-hooks
```
