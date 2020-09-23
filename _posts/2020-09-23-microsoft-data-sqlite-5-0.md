---
layout: post
title: Microsoft.Data.Sqlite 5.0
tags: sqlite
---

Microsoft.Data.Sqlite 5.0 RC1 is available now on [NuGet](https://www.nuget.org/packages/Microsoft.Data.Sqlite/5.0.0-rc.1.20451.13). These are likely the same bits will ship at [.NET Conf](https://www.dotnetconf.net/) on November 10th. Try it today and [let me know](https://github.com/dotnet/efcore/issues/new?labels=area-adonet-sqlite%2C+customer-reported&template=bug_report_sqlite.md) if you run into any issues.

Changes
=======

* Added [conceptual documentation](https://docs.microsoft.com/dotnet/standard/data/sqlite/)
* @nmichels Updated SqliteDataReader.GetBytes, GetChars, and GetTextReader to stream using [SqliteBlob](https://docs.microsoft.com/dotnet/standard/data/sqlite/blob-io) when possible
* @AlexanderTaeschner Enabled executing [EXPLAIN](https://sqlite.org/lang_explain.html) statements without specifying parameters
* @AlexanderTaeschner Added support for [deferred transactions](https://docs.microsoft.com/dotnet/standard/data/sqlite/transactions#deferred-transactions)
* Updated to SQLitePCLRaw 2.0.4
  * Added the SQLitePCLRaw.bundle_sqlite3 [bundle](https://docs.microsoft.com/dotnet/standard/data/sqlite/custom-versions?tabs=netcore-cli#bundles) for using the system version of SQLite
  * Added support for MIPS64 Linux
  * Updated to SQLite 3.33.0
    * Added [generated columns](https://sqlite.org/gencol.html)
    * Added the [UPDATE FROM](https://sqlite.org/lang_update.html#update_from) SQL statement
    * Added the [iif](https://sqlite.org/lang_corefunc.html#iif) SQL function

I also want to take this opportunity to give a huge thank you to the amazing community that has sprung up around this library. Thank you for all your support!
