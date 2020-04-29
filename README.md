# Tusker

A PostgreSQL specific migration tool.

## Elevator pitch

Do you want to write your database schema directly as SQL
which is understood by PostgreSQL?

Do you want to be able to make changes to this schema and
generate the SQL which is required to migrate between the
old and new schema version?

Tusker does exactly this.

## Installation

```shell
pip3 install tusker
```

Now you should be able to run tusker. Give it a try:

```shell
tusker --help
```

## Getting started

Once tusker is installed create a new file called `schema.sql`:

```sql
CREATE TABLE fruit (
    id BIGINT GENERATED DEFAULT AS IDENTIFY,
    name TEXT NOT NULL UNIQUE
);
```

Tusker also needs an empty `migrations` directory:

```shell
mkdir migrations
```

Now you should be able to create your first migration:

```
tusker diff
```

The migration is printed to the console and all you need to do is
copy and paste the output into a new file in the migrations directory.
Alternatively you can also pipe the output of `tusker diff` into the
target file:

```
tusker diff > migrations/0001_initial.sql
```

After that check that your `schema.sql` and your `migrations` are in sync:

```
tusker diff
```

This should give you an empty output. This means that there is no difference
between applying the migrations in order and the target schema.

If you want to change the schema in the future simply change the `schema.sql`
and run `tusker diff` to create the migration for you.

Give it a try and change the `schema.sql`:

```sql
CREATE TABLE fruit (
    id BIGINT GENERATED DEFAULT AS IDENTIFY,
    name TEXT NOT NULL UNIQUE,
    color TEXT NOT NULL DEFAULT ''
);
```

Create a new migration:

```
tusker diff > migrations/0002_fruit_color.sql
```

**Congratulations! You are now using SQL to write your migrations. You are no longer limited by a 3rd party data definition language or an object relational wrapper.**

## Configuration

In order to run tusker you do not need a configuration file. The following
defaults are assumed:

- The file containing your database schema is called `schema.sql`
- The directory containing the migrations is called `migrations`
- Your current user can connect to the database using a unix
  domain socket without a password.

You can also create a configuration file called `tusker.toml`. The default
configuration looks like that:

```toml
[schema]
filename = "schema.sql"

[migrations]
directory = "migrations"

[database]
#host = ""
#port = 5432
#user = ""
#password = ""
dbname = "tusker"
```

Instead of the exploded form of `host`, `port`, etc. it
is also possible to pass a connection URL:

```toml
[schema]
filename = "schema.sql"

[migrations]
directory = "migrations"

[database]
url = "postgresql:///my_awesome_db"
```

## How can I use the generated SQL files?

The resulting SQL files can either be applied to the database by hand
or by using one of the many great tools and libraries which support
applying SQL files in order.

Some recommendations are:

- NodeJS: [marv](https://www.npmjs.com/package/marv)
- Rust: [pgmigrate](https://crates.io/crates/pgmigrate)

## How does it work?

Upon startup `tusker` reads all files from the `migration` directory
and runs them on an empty database. Another empty database is created
and the target schema is created. Then those two schemas are
diffed using the excellent [migra](https://pypi.org/project/migra/)
tool and the output printed to the console.

## FAQ

### Tusker printed an error and it left the temporary tables behind. How
can I remove them?

Run `tusker clean`. This will remove all databases which were created
by previous runs of tusker. Tusker only removes databases which are
marked with a `CREATED BY TUSKER` comment.
