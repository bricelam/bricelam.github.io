---
layout: post
title: Microsoft.Data.Sqlite 3.0
tags: sqlite
---

Version 3.0 of Microsoft.Data.Sqlite is nearly done. [Preview 8][1] is available on NuGet, and the final version will be
available alongside the rest of .NET Core 3.0 at [.NET Conf][2].

Beware, there are a handful of breaking changes in this release that could break your application. Please read these
release notes carefully.

Better database types
---------------------

SQLite has a [dynamic type system][3] with only four intrinsic types--INTEGER (a signed 64-bit integer), REAL (a 64-bit
floating point value), TEXT, and BLOB. This presents several problems when creating an ADO.NET provider. We've had a lot
of feedback in this area and wanted to use version 3.0 to make a few breaking changes to improve type mapping and
inference.

New char mapping (breaking change)
================

`char` values now map to `TEXT`. These were previously mapped to INTEGER values encoded using UTF-16. This made queries
unnecessarily complex and would impose arbitrary restrictions on future UTF-8 capabilities.

You can migrate the data of char columns created using previous version by using SQL like the following.

``` sql
-- Convert char values from INTEGER to TEXT
UPDATE myTable
SET charColumn = char(charColumn)
WHERE typeof(charColumn) = 'integer';
```

Alternatively, you could keep the INTEGER values in the database and update your application code to specify a parameter
type as follows.

``` cs
var command = connection.CreateCommand();
command.CommandText =
@"
    SELECT *
    FROM myTable
    WHERE charColumn = $value
";

// Continue using INTEGER for char
command.Parameters.AddWithValue("$value", 'Y')
    .SqliteType = SqliteType.Integer;
```

New Guid mapping (breaking change)
================

Similarly, `Guid` values also now map to `TEXT`. These were previously mapped to BLOB values, but we've since learned
that there is no standard binary format for these values. This caused problems when accessing a database using other
technologies which interpret the bytes differently.

To migrate your Guid columns, use the following SQL.

``` sql
-- Convert Guid values from BLOB to TEXT
UPDATE myTable
SET guidColumn = hex(substr(guidColumn, 4, 1)) ||
                 hex(substr(guidColumn, 3, 1)) ||
                 hex(substr(guidColumn, 2, 1)) ||
                 hex(substr(guidColumn, 1, 1)) || '-' ||
                 hex(substr(guidColumn, 6, 1)) ||
                 hex(substr(guidColumn, 5, 1)) || '-' ||
                 hex(substr(guidColumn, 8, 1)) ||
                 hex(substr(guidColumn, 7, 1)) || '-' ||
                 hex(substr(guidColumn, 9, 2)) || '-' ||
                 hex(substr(guidColumn, 11, 6))
WHERE typeof(guidColumn) == 'blob';
```

Or, continue using BLOB values by specifying a parameter type:

``` csharp
var command = connection.CreateCommand();
command.CommandText =
@"
    SELECT *
    FROM myTable
    WHERE guidColumn = $value
";

// Continue using BLOB for Guid
command.Parameters.AddWithValue("$value", new Guid())
    .SqliteType = SqliteType.Blob;
```

Better inference
================

Because of SQLite's dynamic type system, there are a few places where a column's database type isn't available but
ADO.NET requires us to return something. We've improved how we infer database types in these situations and have
standardized on returning BLOB when we simply don't know.

Other breaking changes
----------------------

There are a few other minor breaking changes in this release.

We updated to [SQLitePCL.raw version 2.0][4]. This is the low-level API maintained by @ericsink that we build on top of
to call into the native SQLite library. Most applications probably aren't using it directly, but if you are, see the
release notes for breaking changes.

The Microsoft.Data.Sqlite package now depends on SQLitePCLRaw.bundle_e_sqlite. This only affects **Xamarin.iOS** which
will now use a version of SQLite consistent with all the other target frameworks and runtimes.

SqliteCommand.CommandTimeout has been updated so that a value of **zero means no timeout**. Previously, it directed
commands to timeout immediately when the database was busy. The new behavior is consistent with ADO.NET recommendations.

In order to fix [a bug][5] with batched statements, we had to change where **timeout exceptions** could be thrown from.
In addition to the execute methods on SqliteConnection, these exceptions can now also be thrown from NextResult and
Dispose on SqliteDataReader.

Re-opening a connection
-----------------------

In previous versions of Microsoft.Data.Sqlite, things like foreign key enforcement, user-defined functions, and SQLite
extensions were cleared when a connection was closed. While this reflected the underlying behavior of SQLite, it
hindered the usability of Microsoft.Data.Sqlite.

In version 3.0, we now preserve state between connection close and re-open. Calls the following methods on
SqliteConnection will remain effective for the lifetime of the connection object.

* CreateAggregate
* CreateCollation
* CreateFunction
* EnableExtensions

We also added a method on SqliteConnection for loading extensions so they too can be preserved.

* LoadExtension

Finally, we added new **connection string keywords** to help maintain consistent behavior between connection close and
re-open.

| Keyword            | Default | Description                                                                           |
| ------------------ | ------- | ------------------------------------------------------------------------------------- |
| Foreign Keys       | null    | When true or false, sends `PRAGMA foreign_keys` immediately after opening a connection. When null, no pragma is sent. |
| Password           | N/A     | When specified, sends `PRAGMA key` on connection open.                                |
| Recursive Triggers | false   | When true, sends `PRAGMA recursive_triggers` on connection open. When false, no pragma is sent. |

Blob I/O
--------

@AlexanderTaeschner kept up his contribution streak and enabled streaming values into and out of a database. This
feature of SQLite can reduce the amount of memory used by your application. It's particularly useful when parsing or
transforming large amounts of data.

Use the new `SqliteBlob` type to stream values into a database. This type inherits from Stream and works with all the
existing Stream goodness in .NET. Here is an example.

``` csharp
// Insert a row to hold the data
var command = connection.CreateCommand();
command.CommandText =
@"
    INSERT INTO myTable(blobColumn)
    VALUES (zeroblob($length));

    SELECT last_insert_rowid();
";
command.Parameters.AddWithValue("$length", inputStream.Length);
var rowid = (long)command.ExecuteScalar();

// Open a stream to write the data
using (var blobStream = new SqliteBlob(connection, "myTable", "blobColumn", rowid))
{
    await inputStream.CopyToAsync(blobStream);
}
```

To stream values out of a database, use `SqliteDataReader.GetStream()`.

``` csharp
// NB: Must select rowid (or an alias)
var command = connection.CreateCommand();
command.CommandText =
@"
    SELECT rowid, blobColumn
    FROM myTable
    WHERE rowid = $rowid
";
command.Parameters.AddWithValue("$rowid", rowid);

using (var reader = command.ExecuteReader())
{
    if (reader.Read())
    {
        using (var blobStream = reader.GetStream(1))
        {
            // TODO: Use blobStream
        }
    }
}
```

Feedback
--------

Please try out Microsoft.Data.Sqlite version 3.0 while it's still in preview. [Let us know][6] if you find any issues or
otherwise have feedback about the library.


  [1]: https://www.nuget.org/packages/Microsoft.Data.SQLite/3.0.0-preview8.19405.11
  [2]: https://www.dotnetconf.net/
  [3]: https://www.sqlite.org/datatype3.html
  [4]: https://github.com/ericsink/SQLitePCL.raw/blob/master/v2.md
  [5]: https://github.com/aspnet/EntityFrameworkCore/issues/13830
  [6]: https://github.com/aspnet/EntityFrameworkCore/issues/new
