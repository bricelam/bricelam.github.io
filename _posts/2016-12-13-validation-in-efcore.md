---
layout: post
title: Validation in EF Core
tags: entity-framework
---

In Entity Framework 4.1+ we would validate entities before sending them to the database. See [Entity Framework
Validation][1] to read more about it.

We don't perform validation in EF Core, but there is a quick way to add at least some of it back. You can override
`SaveChanges` and use the `Validator`. Here is some code.

```csharp
class MyContext : DbContext
{
    public override int SaveChanges()
    {
        var entities = from e in ChangeTracker.Entries()
                       where e.State == EntityState.Added
                           || e.State == EntityState.Modified
                       select e.Entity;
        foreach (var entity in entities)
        {
            var validationContext = new ValidationContext(entity);
            Validator.ValidateObject(entity, validationContext);
        }

        return base.SaveChanges();
    }
}
```

Now, when `SaveChanges` is called, any `ValidationAttribute` (e.g. `[Required]` and `[StringLength]`) or
`IValidatableObject` on your entity type classes will be checked before sending the `INSERT` or `UPDATE` statements to
the database.


  [1]: https://msdn.microsoft.com/en-us/library/gg193959.aspx
