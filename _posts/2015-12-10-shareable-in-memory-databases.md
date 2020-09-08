---
layout: post
title: Shareable In-Memory SQLite Databases
tags: sqlite
---

I recently added a feature to [Microsoft.Data.Sqlite][1] to enable [shareable in-memory databases][2]. These are in-memory
databases that can be accessed by multiple connections. Just remember to keep at least one connection open or the database will
disappear.

Here is an example demonstrating how you can use them.

```cs

// Databases are shared by using the same Data Source name
var connectionString = "Data Source=sharedmemdb;Mode=Memory;Cache=Shared";

// The first connection controls the lifetime of the database. When it's closed, the
// database is deleted
using (var connection1 = new SqliteConnection(connectionString))
{
    connection1.Open();

    var command1 = connection1.CreateCommand();
    command1.CommandText =
        "CREATE TABLE Message ( Text TEXT );" +
        "INSERT INTO Message ( Text ) VALUES ( 'Is there anybody out there?' );";
    command1.ExecuteNonQuery();

    // Additional connections can be made
    using (var connection2 = new SqliteConnection(connectionString))
    {
        connection2.Open();

        var command2 = connection2.CreateCommand();
        command2.CommandText = "SELECT Text FROM Message;"

        // This will return the data inserted on connection1
        var message = command2.ExecuteScalar() as string;
    }
}
```


  [1]: https://github.com/aspnet/Microsoft.Data.Sqlite
  [2]: https://www.sqlite.org/inmemorydb.html#sharedmemdb
