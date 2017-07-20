---
layout: post
title: Bulk Insert in Microsoft.Data.Sqlite
tags: sqlite
---

You've got a large set of data to import into your SQLite database, but you can't find the bulk insert API in
[Microsoft.Data.Sqlite][1]. That's because there isn't one! SQLite doesn't have any special way to bulk insert data.

The two things you can to do to speed up inserts are:

1. Use a transaction.
2. Re-use the same `INSERT` command.

```csharp
using (var transaction = connection.BeginTransaction())
using (var command = connection.CreateCommand())
{
    command.CommandText =
        "INSERT INTO contact(name, email) " +
        "VALUES($name, $email);";

    var nameParameter = command.CreateParameter();
    nameParameter.ParameterName = "$name";
    command.Parameters.Add(nameParameter);

    var emailParameter = command.CreateParameter();
    emailParameter.ParameterName = "$email";
    command.Parameters.Add(emailParameter);

    foreach (var contact in contacts)
    {
        nameParameter.Value = contact.Name;
        emailParameter.Value = contact.Email;
        command.ExecuteNonQuery();
    }

    transaction.Commit();
}
```

Faster in 2.1
-------------
In Microsoft.Data.Sqlite version 2.1.0 this code will get *even faster* thanks to a contribution by @AlexanderTaeschner.
He implemented `SqliteCommand.Prepare()` which allows you to precompile a command. But even if you don't call the
method, subsequent executions will reuse the compilation of the first one. My initial benchmarking indicates up to 4x
more inserts per second in some cases!

Dangerously Fast
----------------
If you're willing to take risks, you can squeeze even more speed out of it by using a couple `PRAGMA` statements. Make
sure, however, that you understand [the consequences][2] of doing this.

```sql
PRAGMA journal_mode = MEMORY;
PRAGMA synchronous = OFF;
```


  [1]: https://github.com/aspnet/Microsoft.Data.Sqlite
  [2]: https://www.sqlite.org/howtocorrupt.html#cfgerr
