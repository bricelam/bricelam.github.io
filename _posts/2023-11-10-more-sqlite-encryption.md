---
layout: post
title: More SQLite Encryption in .NET
tags: sqlite
---
Recently, @utelle, the author of [SQLite3 Multiple Ciphers](https://utelle.github.io/SQLite3MultipleCiphers/) reached out to me about creating a new [SQLitePCLRaw](https://github.com/ericsink/SQLitePCL.raw) bundle. We worked together with @ericsink to make it part of the main SQLitePCLRaw project. Today, I'm super excited to announce the initial preview of [SQLitePCLRaw.bundle_e_sqlite3mc](https://www.nuget.org/packages/SQLitePCLRaw.bundle_e_sqlite3mc). But before we talk about that, let's talk a little about what SQLite3 Multiple Ciphers is.

SQLite3 Multiple Ciphers is an extension to SQLite for reading and writing encrypted databases. It supports five different encryption schemes including the ones for [System.Data.SQLite](https://system.data.sqlite.org/index.html/doc/trunk/www/index.wiki), [SQLCipher](https://www.zetetic.net/sqlcipher/), and [wxSQLite3](https://github.com/utelle/wxsqlite3). It's also cross-platform which means it can be used with Linux, macOS, Windows, Android, and iOS.

The new package makes it super easy to use with various .NET libraries.

# Microsoft.Data.Sqlite + Dapper

Start using it with [Microsoft.Data.Sqlite](https://learn.microsoft.com/dotnet/standard/data/sqlite/) and [Dapper](https://dapperlib.github.io/Dapper/) by installing the right packages. Be sure to use the package ending in **.Core** and not the main Microsoft.Data.Sqlite once. This avoids installing two conflicting bundles into your project.

```sh
dotnet add package Microsoft.Data.Sqlite.Core
dotnet add package SQLitePCLRaw.bundle_e_sqlite3mc
dotnet add package Dapper
```

After that, you can simply use the `Password` keyword in your connection string to create and open an encrypted database. This uses the default encryption scheme.

```cs
using Dapper;
using Microsoft.Data.Sqlite;

using var connection = new SqliteConnection("Data Source=example.db;Password=Password12!");
var version = connection.ExecuteScalar<string>("select sqlite3mc_version()");
Console.WriteLine(version);
```

# SQLite-net

For [SQLite-net](https://github.com/praeclarum/sqlite-net), be sure to use the `sqlite-net-base` package instead of the main one to avoid conflicting bundles.

```sh
dotnet add package sqlite-net-base
dotnet add package SQLitePCLRaw.bundle_e_sqlite3mc
```

SQLite-net has a convenient `key` parameter you can pass to the connection string.

```cs
using SQLite;

SQLitePCL.Batteries_V2.Init();

var connection = new SQLiteConnection(new("example.db", storeDateTimeAsTicks: true, key: "Password12!"));
var version = connection.ExecuteScalar<string>("select sqlite3mc_version()");
Console.WriteLine(version);
```

# EF Core

On [EF Core](https://learn.microsoft.com/ef/core/), again, use the package ending in **.Core** to avoid conflicting bundles.

```sh
dotnet add package Microsoft.EntityFrameworkCore.Sqlite.Core
dotnet add package SQLitePCLRaw.bundle_e_sqlite3mc
```

Under the covers, EF Core uses Microsoft.Data.Sqlite, so again, you can just specify the `Password` keyword in the connection string.

```cs
options.UseSqlite("Data Source=example.db;Password=Password12!");
```

# Existing Databases

At the beginning, I mentioned SQLite3 Multiple Ciphers supports multiple encryption schemes. Let's look at how to configure the scheme.

## System.Data.SQLite

If you have an existing database encrypted by System.Data.SQLite, you can open it by using a URI filename in your connection string and specifying the `rc4` cipher.

```txt
Data Source=file:example.db?cipher=rc4;Password=Password12!
```

It looks a little funny, but it should work. See [the docs](https://utelle.github.io/SQLite3MultipleCiphers/docs/ciphers/cipher_sds_rc4/) for additional options that can be specified in the URI.

I think being able to read encrypted databases created by System.Data.SQLite will unblock several projects that have been wanting to move to Microsoft.Data.Sqlite or EF Core and go cross-platform.

## SQLCipher

If you have a database created by SQLCipher (including `SQLitePCLRaw.bundle_e_sqlcipher`), you can open it by specifying `sqlcipher` in the connection string.

```txt
Data Source=file:example.db?cipher=sqlcipher&legacy=4;Password=Password12!
```

SQLite3 Multiple Ciphers is not intended to be a drop-in replacement for SQLCipher, but it's a great tool for working with existing SQLCipher databases.

The `legacy` option is just to avoid breaks in the future when SQLCipher changes their defaults. Again, be sure to check [the docs](https://utelle.github.io/SQLite3MultipleCiphers/docs/ciphers/cipher_sqlcipher/) for additional options that can be specified.

# Feedback

I'm really excited to have a new encryption option for SQLite in .NET. Please give it a try and [let us know](https://github.com/ericsink/SQLitePCL.raw/issues) if you run into any issues.
