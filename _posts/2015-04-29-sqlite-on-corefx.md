---
layout: post
title: SQLite on .NET Core
tags: sqlite
---

One weekend back in February of 2014, I had the crazy idea to start implementing a SQLite ADO.NET provider as a portable class
library. My initial goal was to better understand SQLite, ADO.NET, and unmanaged interop. I showed it to my team (the Entity
Framework team), and we decided that it was strategically important to running EF on Windows Store and Windows Phone. My code
eventually evolved into the [Microsoft.Data.Sqlite][1] package.

Usage
=====
The provider is built on top of the [System.Data.Common][2] contract. This contract is a very small subset of the ADO.NET provider
model. Using the provider should feel very natural to anyone familiar with ADO.NET.

{% highlight csharp %}
using (var connection = new SqliteConnection("" +
    new SqliteConnectionStringBuilder
    {
        DataSource = "hello.db"
    }))
{
    connection.Open();

    using (var transaction = connection.BeginTransaction())
    {
        var insertCommand = connection.CreateCommand();
        insertCommand.Transaction = transaction;
        insertCommand.CommandText = "INSERT INTO message ( text ) VALUES ( $text )";
        insertCommand.Parameters.AddWithValue("$text", "Hello, World!");
        insertCommand.ExecuteNonQuery();

        var selectCommand = connection.CreateCommand();
        selectCommand.Transaction = transaction;
        selectCommand.CommandText = "SELECT text FROM message";
        using (var reader = selectCommand.ExecuteReader())
        {
            while (reader.Read())
            {
                var message = reader.GetString(0);
                Console.WriteLine(message);
            }
        }

        transaction.Commit();
    }
}
{% endhighlight %}

Batching
--------
The only real feature that the library adds to the native SQLite interfaces is batching. The native interfaces only support
compiling and executing one statement at a time. This library implements batching in a way that should feel completely
transparent. Here is an example of using batching.

{% highlight csharp %}
using (var connection = new SqliteConnection("Data Source=hello.db"))
{
    var command = connection.CreateCommand();
    command.CommandText =
        "UPDATE message SET text = $text1 WHERE id = 1;" +
        "UPDATE message SET text = $text2 WHERE id = 2";
    command.Parameters.AddWithValue("$text1", "Hello");
    command.Parameters.AddWithValue("$text2", "World");

    connection.Open();
    command.ExecuteNonQuery();
}
{% endhighlight %}

Platforms
=========
Currently, Microsoft.Data.Sqlite works on the following platforms.

- .NET Framework 4.5
- Mono 3
- .NET Core 5
    - .NET Native
    - DNX Core 5

We also intend to add support for these frameworks.

- Xamarin
- Windows Universal

Yet another...
==============
With so many other great frameworks like [System.Data.SQLite][3], [Mono.Data.Sqlite][4], [sqlite-net][5], [Portable Class Library
for SQLite][6], [SQLitePCL.raw][7], and more why create yet another one? The differentiating feature of [Microsoft.Data.Sqlite][1] is
that it implements the [System.Data.Common][2] contract which is built on top of [.NET Core][8].


  [1]: https://github.com/aspnet/DataCommon.SQLite
  [2]: http://www.nuget.org/packages/System.Data.Common
  [3]: http://system.data.sqlite.org
  [4]: http://www.mono-project.com/docs/database-access/providers/sqlite/
  [5]: https://github.com/praeclarum/sqlite-net
  [6]: http://sqlitepcl.codeplex.com/
  [7]: https://github.com/ericsink/SQLitePCL.raw
  [8]: http://blogs.msdn.com/b/dotnet/archive/2014/12/04/introducing-net-core.aspx
