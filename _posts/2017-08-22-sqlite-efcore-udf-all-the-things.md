---
layout: post
title: 'SQLite & EF Core: UDF all the things!'
tags: [ entity-framework, sqlite ]
---

We recently had two community contributions for using user-defined functions (UDFs) in EF Core and Microsoft.Data.Sqlite
that synergize well together. I thought I'd write a sarcastic blog post showing you how you can make everything a UDF
whether or not it should be.

Without UDFs
============
Here is our starting code. We store `People` entities with a `Name` property in our database.

```cs
class Person
{
    [Key]
    public string Name { get; set; }
}
```

Our `DbContext` class allows us to interact with the SQLite database. It has a helper method we use to greet people.

```cs
class HelloContext : DbContext
{
    public DbSet<Person> People { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=Hello.db");

    public static string Greet(string name)
        => $"Hello, {name}.";
}
```

We use this query to greet all the users.

```cs
var greetings = Enumerable.ToList(
    from p in db.People
    select new { Value = HelloContext.Greet(p.Name) });
```

EF Core translates whatever parts of the query it can into SQL and performs client-side evaluation on the rest. For us,
this means the following SQL is sent to the database.

```sql
SELECT "p"."Name"
FROM "People" AS "p"
```

This retrieves the names of each user from the database then EF Core passes them to our `HelloContext.Greet()` method in
order to return the greetings.

<p style="text-align:center"><img src="{{ "/attachments/WithoutUDF.png" | absolute_url }}" alt="Without UDF Sequence Diagram"></p>

With UDFs
=========
You explain this to your DBA friend, but he freaks out. Everyone knows that processing things on the databases is
faster, :trollface: so you decide to create a user-defined function to replace the `HelloContext.Greet()` method.

To get EF Core to generate the appropriate SQL, simply add the `[DbFunction]` attribute. This feature was recently
contributed by @pmiddleton.

```cs
[DbFunction("greet")]
public static string Greet(string name)
    => $"Hello, {name}.";
```

Now, the following SQL is sent to the database. The whole query will be evaluated on the server!

```sql
SELECT greet("p"."Name") AS "Value"
FROM "People" AS "p"
```

But how do we define the function in the database? Most databases have a special SQL-based language to define functions
(e.g. Transact-SQL or PL/SQL). SQLite, however, isn't like most databases. It runs in-process with your application, so
the most logical language for them to use was to use was whatever language your application uses. In our case, C#!

To register a UDF with SQLite, use the `SqliteConnection.CreateFunction()` method. This awesome, and very .NET-friendly
API was contributed by @AlexanderTaeschner.

Since UDF registrations in SQLite only persists until the connection is closed, we also need to modify our `DbContext`
a bit.

```cs
protected override void OnConfiguring(DbContextOptionsBuilder options)
{
    var connection = new SqliteConnection("Data Source=Hello.db");
    connection.Open();

    connection.CreateFunction("greet", (Func<string, string>)Greet);

    // Passing in an already open connection will keep the connection open between
    // requests.
    options.UseSqlite(connection);
}
```

Ta-da! Everything is evaluated on the server. When SQLite comes across the `greet()` function in SQL, it will invoke a
function pointer that marshals values into a managed delegate that calls our `HelloContext.Greet()` method then marshals
the result back to native so Microsoft.Data.Sqlite can marshal it back to managed and pass it to EF Core. (Yes, this is,
in fact, *less* performant for our particular scenario.)

<p style="text-align:center"><img src="{{ "/attachments/WithUDF.png" | absolute_url }}" alt="With UDF Sequence Diagram"></p>

When to UDF
===========
Despite being totally awesome, using a UDF in this example is obviously not the way to go. It adds a whole lot of
complexity to something that was already working well.

However, when a UDF results in *less data* being returned from the database, you can see significant benefits. For
example, if an expression in the `WHERE` clause can't be translated, the whole row still needs to be returned from the
database even if the client just reads one field before filtering it out of the final results.

Pre-release
===========
Note, the `SqliteConnection.CreateFunction()` method is scheduled to be released with Microsoft.Data.Sqlite version
2.1.0. If you want to try it out sooner, you can always use the [nightly builds][1].


  [1]: https://dotnet.myget.org/feed/aspnetcore-dev/package/nuget/Microsoft.Data.Sqlite
