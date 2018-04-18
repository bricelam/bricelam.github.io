---
layout: post
title: Pluralization in EF Core
tags: entity-framework
---

We've been improving design-time extensibility in EF Core 2.1. One new feature is the ability for NuGet packages to
register design-time services. As a reference implementation for how to do this, I've created the
[bricelam\EFCore.Pluralizer][1] repo. While the repo primarily serves as a sample for anyone who wants to create
design-time extensions for EF Core, anyone that uses EF Core with an existing database might find the
[Bricelam.EntityFrameworkCore.Pluralizer][2] package useful.

This package is a port of the pluralizer used by EF6 when reverse engineering a model from andatabase. To use it, simply
install it.

To install and use the package inside Visual Studio using NuGet's Package Manager Console (PMC), type the following.

``` powershell
Install-Package Bricelam.EntityFrameworkCore.Pluralizer -Version 1.0.0-rc2
Scaffold-DbContext 'Filename=chinook.db' Microsoft.EntityFrameworkCore.Sqlite
```

If you're on a command-line instead, type this.

``` shell
dotnet add package Bricelam.EntityFrameworkCore.Pluralizer --version 1.0.0-rc2
dotnet ef dbcontext scaffold "Filename=chinook.db" Microsoft.EntityFrameworkCore.Sqlite
```

After installing it, you'll get pluralized `DbSet` property names...

``` csharp
class ChinookContext : DbContext
{
    public DbSet<Artist> Artists { get; set; }

    // ...
}
```

...and pluralized collection navigation property names.

``` csharp
class Artist
{
    // ...

    public ICollection<Album> Albums { get; set; }
}
```

It will also singularize entity type class names if you happen to have pluralized table names in your database.


  [1]: https://github.com/bricelam/EFCore.Pluralizer
  [2]: https://www.nuget.org/packages/Bricelam.EntityFrameworkCore.Pluralizer
