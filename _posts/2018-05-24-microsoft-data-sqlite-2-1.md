---
layout: post
title: Announcing Microsoft.Data.Sqlite 2.1
tags: sqlite
---

Thanks to some amazing contributions by @AlexanderTaeschner, version 2.1 of Microsoft.Data.Sqlite turned into a
feature-packed release!

SQLitePCL.raw 1.1.11
====================
We've updated our dependency on SQLitePCL.raw to version 1.1.11. Some of the great features added by @ericksink since
the previous version we depended on (1.1.7) are:

* SQLite was updated from 3.18.2 to version 3.22.0
* The FTS5 extension was enabled
* Additional runtimes were supported:
  * linux-arm
  * linux-arm64
  * linux-armel
  * linux-musl-x64
  * win8-arm

Prepared statements
===================
We enabled `SqliteCommand.Prepare()`, but you don't actually have to call it. Once a command is executed, the
compilation of its SQL statements gets reused by subsequent executions. This can result in large performence
improvements. See my post about [bulk inserts][1] for an example that can take advantage of this feature.

User-defined functions
======================
User-defined functions can now be created by using the `SqliteConnection.CreateFunction()` and `CreateAggregate()`
overloads. For example, you can create a scalar function to calculate the volume of a cylinder.

```cs
connection.CreateFunction(
    "volume",
    (double radius, double height)
        => Math.PI * Math.Pow(radius, 2) * height);
```

And use the function in SQL to find the biggest cylinder.

```sql
SELECT id, volume(radius, height) AS volume
FROM cylinder
ORDER BY volume DESC
LIMIT 1
```

SQLite will evaluate the function by invoke the .NET delegate. You can even set a breakpoint to debug it!

For more examples, see the [aggregate function][2] and [regular expression][3] samples.

This feature also pairs nicely with EF Core's `[DbFunction]` attribute. See my post [SQLite & EF Core: UDF all the
things!][4]

Custom collations
=================
Collating sequences are used to compare strings. SQLite has a built-in NOCASE collation you can use to perform
case-insensitive comparisons.

```sql
SELECT 'Λ' = 'λ' COLLATE NOCASE;
```
Unfortunately, it only works with the ASCII characters A through Z. With the `CreateCollation()` method on
`SqliteConnection`, you can now define your own (or redefine existing ones).

```cs
connection.CreateCollation(
    "NOCASE",
    (x, y) => string.Compare(x, y, ignoreCase: true));
```

This feature was actually added in version 2.0, but thought it deserved to be called out again here.

Result metadata
===============
`SqliteDataReader.GetSchemaTable()` can now be used to retrieve metadata about the columns in a result including the
source of the data. The API returns a table with the following columns.

| Column           | Type   | Description                              |
| ---------------- | ------ | ---------------------------------------- |
| AllowDBNull      | bool   | If the column can be NULL                |
| BaseCatalogName  | string | The database name                        |
| BaseColumnName   | string | The name of the column in the table      |
| BaseTableName    | string | The table name                           |
| ColumnName       | string | The name of the column in the result     |
| ColumnOrdinal    | int    | The rank of the column within the result |
| DataType         | Type   | The CLR type of the column               |
| DataTypeName     | string | The SQL type of the column               |
| IsAliased        | bool   | If the column is aliased                 |
| IsAutoIncrement  | bool   | If the column is auto-increment          |
| IsExpression     | bool   | If the column is an expression           |
| IsKey            | bool   | If the column is part of the primary key |
| IsUnique         | bool   | If the column is unique                  |

See the [result metadata sample][5] for an example of using this API.

Value coercion
==============
Values can now be coerced into alternative types by setting `SqliteParameter.SqliteType`. The following alternatives are
allowed.

| .NET     | SQL  | Description               |
| -------- | ---- | ------------------------- |
| Char     | TEXT | A one-character string    |
| DateTime | REAL | The Julian Day value      |
| Guid     | TEXT | The string representation |
| TimeSpan | REAL | The total days            |

The values are also transparently coerced back to the original type when calling the corresponding method on
`SqliteDataReader`.

See the [date and time sample][6] for an example of some functionality this enables.

The little things
=================
There are also a handful of other APIs added or enabled in this release:

* `DbProviderFactories.GetFactory(DbConnection)` now works when passed a `SqliteConnection` object.
* `SqliteConnection`
  * `DefaultTimeout` sets the timeout used by implicilty created commands. (e.g. `BeginTransaction()`)
  * `BackupDatabase()` copies the current database to another one.
* `SqliteDataReader`
  * `GetDateTimeOffset()` and `GetTimeSpan()` were added for completeness.
  * `GetBytes()`, `GetChars()`, and `GetStream()` work now. See issue [#18][7] for our plans enhance them.
* `SqliteException.SqliteExtendedErrorCode` gives you the [extended result code][8] of an error.
* `SqliteParameter.Size` can now be used to truncate `string` and `byte[]` values.


  [1]: {% post_url 2017-07-20-sqlite-bulk-insert %}
  [2]: https://github.com/aspnet/Microsoft.Data.Sqlite/blob/2.1.0/samples/AggregateFunctionSample/Program.cs
  [3]: https://github.com/aspnet/Microsoft.Data.Sqlite/blob/2.1.0/samples/RegularExpressionSample/Program.cs
  [4]: {% post_url 2017-08-22-sqlite-efcore-udf-all-the-things %}
  [5]: https://github.com/aspnet/Microsoft.Data.Sqlite/blob/2.1.0/samples/ResultMetadataSample/Program.cs
  [6]: https://github.com/aspnet/Microsoft.Data.Sqlite/blob/2.1.0/samples/DateAndTimeSample/Program.cs
  [7]: https://github.com/aspnet/Microsoft.Data.Sqlite/issues/18
  [8]: https://www.sqlite.org/rescode.html#extrc
