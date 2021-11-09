---
layout: post
title: Microsoft.Data.Sqlite 6
tags: sqlite
---

It's that time of year again. [Microsoft.Data.Sqlite](https://docs.microsoft.com/dotnet/standard/data/sqlite/) version 6.0.0 has been released today alongside the rest of .NET 6. Update now and [let us know](https://github.com/dotnet/efcore/issues/new?assignees=&labels=area-adonet-sqlite%2C+customer-reported&template=bug_report_sqlite.md) if you run into any issues.

Connection Pooling
==================

The underlying, native SQLite connections are now pooled by default. This greatly improves the performance of opening and closing connections. This is especially noticeable in scenarios where opening the underlying connection is expensive as is the case when [using encryption](https://docs.microsoft.com/dotnet/standard/data/sqlite/encryption) or in scenarios where there are lots of short-lived connections to the database. The [Orchard Core](https://orchardcore.net/) benchmarks went from 5.5K to 14.5K requests per second with the latest version of Microsoft.Data.Sqlite.

Beware, however, that the database file may still be locked after you close a connection. If this becomes a problem, you can manually clear the pool to release the lock:

```cs
SqliteConnection.ClearPool(connection);

// or

SqliteConnection.ClearAllPools();
```

If you run into any issues, you can turn off connection pooling by specifying `Pooling=False` in your connection string. Please be sure to [file an issue](https://github.com/dotnet/efcore/issues/new?assignees=&labels=area-adonet-sqlite%2C+customer-reported&template=bug_report_sqlite.md) too!

Savepoints
==========

This release implements the ADO.NET Savepoints API. Savepoints enable nested transactions. For more information and a sample, see the new [Savepoints](https://docs.microsoft.com/dotnet/standard/data/sqlite/transactions#savepoints) section in the docs.

DateOnly & TimeOnly
=====================

.NET 6 added two [new types for working with date and time](https://devblogs.microsoft.com/dotnet/date-time-and-time-zone-enhancements-in-net-6/#introducing-the-dateonly-and-timeonly-types) values. These types work just as you'd expect in parameters, data readers, and [user-defined functions](https://docs.microsoft.com/dotnet/standard/data/sqlite/user-defined-functions).

The little things
=================

Here are a few more minor changes also worth mentioning:

* Added the `Command Timeout` [connection string](https://docs.microsoft.com/dotnet/standard/data/sqlite/connection-strings) keyword as part of an effort to standardize it across providers
* Implemented the `Span` overloads of SqliteBlob to avoid allocations
* Removed other unnecessary allocations and conversions between UTF-8 to further improve performance

Happy coding! Don't forget to [vote on the issues](https://github.com/dotnet/efcore/issues?q=is%3Aopen+is%3Aissue+label%3Aarea-adonet-sqlite+sort%3Acreated-asc) you'd like to see implemented in a future release.
