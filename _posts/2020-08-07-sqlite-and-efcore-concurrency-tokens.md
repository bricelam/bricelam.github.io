---
layout: post
title: SQLite and EF Core Concurrency Tokens
tags: [ entity-framework, sqlite ]
---

Entity Framework Core has great built-in support for [optimistic concurrency control](https://docs.microsoft.com/ef/core/saving/concurrency). The best way to utilize this on SQL Server is via a [rowversion](https://docs.microsoft.com/ef/core/modeling/concurrency?tabs=data-annotations#timestamprowversion) column. Unfortunately, SQLite has no such feature. This post show how to implement similar functionality using a trigger.

Start by adding a `Version` property to your entity type to serve as the concurrency token.

``` cs
class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Version { get; set; }
}
```

Configure the property as a concurrency token whose value is generated on add and update. We'll use a default constraint to generate the value when an entity is added. The IsRowVersion method is just shorthand for ValueGeneratedOnAddOrUpdate and IsConcurrencyToken.

``` cs
modelBuilder
    .Entity<Customer>()
        .Property(c => c.Version)
            .HasDefaultValue(0)
            .IsRowVersion();
```

Next, create a trigger to update the value whenever an entity is updated. If you're using [Migrations](https://docs.microsoft.com/ef/core/managing-schemas/migrations/), you can add this to the `Up` method of a new migration using `migrationBuilder.Sql()`. If you're using EnsureCreated, you can create it using `dbContext.Database.ExecuteSqlCommand()` whenever EnsureCreated returns `true`.

``` sql
CREATE TRIGGER UpdateCustomerVersion
AFTER UPDATE ON Customers
BEGIN
    UPDATE Customers
    SET Version = Version + 1
    WHERE rowid = NEW.rowid;
END;
```

That's it! DbUpdateConcurrencyException should now be thrown whenever a concurrent update occurs.

``` cs
using (var db = new MyDbContext())
{
    var customer = db.Customers.Find(1);

    // Simulate a concurrent update
    using (var concurrentDb = new MyDbContext())
    {
        var concurrentCustomer = concurrentDb.Customers.Find(1);
        concurrentCustomer.Name = "David";
        concurrentDb.SaveChanges();
    }

    // Throws DbUpdateConcurrencyException
    customer.Name = "Henry";
    db.SaveChanges();
}
```
