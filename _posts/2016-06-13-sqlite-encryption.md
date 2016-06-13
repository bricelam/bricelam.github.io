---
layout: post
title: Encryption in Microsoft.Data.Sqlite
tags: sqlite
---

One of the frequently asked questions about [Microsoft.Data.Sqlite][1] is: How do I encrypt a database? I think that one
of the main reasons for this is because [System.Data.SQLite][2] comes with an unsupported, Windows-only encryption codec
that can be used by specifying `Password` (or `HexPassword`) in the connection string. The official releases of SQLite,
however, don't come with encryption.

[SEE][3], [SQLCipher][4], [SQLiteCrypt][5] & [wxSQLite3][6] are just some of the solutions I've found that can encrypt
SQLite database files. They all seem to follow the same general pattern, so this post applies to all of them.

Bring Your Own Library
----------------------
The first step is to add a version of the native `sqlite3` library to you application that has encryption.
`Microsoft.Data.Sqlite` comes with the official release of SQLite, but you can copy over `sqlite3.dll` (or
`libsqlite3.so` or `.dylib`) with a custom version.

Do this in Visual Studio by adding it to your project and setting **Copy to Output Directory** to **Copy if newer** in
the item's **Properties**.

Specify the Key
---------------
To enable encryption, Specify the key immediately after opening the connection. Do this by issuing a `PRAGMA key` 
statement. It may be specified as either a string or BLOB literal. SQLite, unfortunately, doesn't support parameters in
`PRAGMA` statements. :disappointed:

```csharp
connection.Open();

var command = connection.CreateCommand();
command.CommandText = "PRAGMA key = 'password';";
command.ExecuteNonQuery();

// Interact with the database here
```

Rekey the Database
------------------
If you want to change the encryption key of a database, issue a `PRAGMA rekey` statement. To decrypt the database,
specify `NULL`.

```csharp
var command = connection.CreateCommand();
command.CommandText = "PRAGMA rekey = 'new-password';";
command.ExecuteNonQuery();
```

  [1]: https://github.com/aspnet/Microsoft.Data.Sqlite
  [2]: http://system.data.sqlite.org/index.html/doc/trunk/www/index.wiki
  [3]: http://www.hwaci.com/sw/sqlite/see.html
  [4]: https://www.zetetic.net/sqlcipher/
  [5]: http://sqlite-crypt.com/index.htm
  [6]: https://github.com/utelle/wxsqlite3
