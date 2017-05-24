---
layout: post
title: 'EF Core Migrations: .NET CLI Commands'
tags: entity-framework
---

One of the [new platforms][1] that we're targeting in Entity Framework Core is [ASP.NET Core][2]. With this new platform
comes a new set of challenges for how we enable the Entity Framework commands. Ever since Entity Framework 4.3, we've
provided a set of PowerShell commands that could be run in Visual Studio from [NuGet's Package Manager Console][3].
However, that won't help you if you're developing on OSX where neither PowerShell nor Visual Studio are available. This
post will show you how to use the new Entity Framework .NET Core CLI Commands.

Installing
==========
In EF Core, we're striving for a more modular design. This has resulted in more fine-grained NuGet packages. The new
Entity Framework commands are contained in the Microsoft.EntityFrameworkCore.Tools package. To use it, add it as a tool to your
project.json file. You'll also need to include Microsoft.EntityFrameworkCore.Design as a build-time dependency.

```json
{
    "dependencies": {
        "Microsoft.EntityFrameworkCore.Design": {
            "version": "1.0.0-*",
            "type": "build"
        }
    },
    "tools": {
        "Microsoft.EntityFrameworkCore.Tools": "1.0.0-*"
    }
}
```

Once you've done this, you should be able to restore packages, change directory into your project (the directory
containing the project.json file), and run the command.

```
dotnet restore
cd src\MyProject
dotnet ef
```

![Entity Framework .NET Core CLI Commands]({{ "/attachments/EFCommands.png" | absolute_url }})

Using
=====
To see what sub-commands are available for the `migrations` command, type `dotnet ef migrations --help`.

To see the usage of the `add` command, type `dotnet ef migrations add --help`.

To add a new migration, type `dotnet ef migrations add MyMigration`.

Here is a full list of the commands currently available.

* `database`
    * `update`--Updates the database to a specified migration
* `dbcontext`
    * `list`--List your DbContext types
    * `scaffold`--Scaffolds a DbContext and entity type classes for a specified database
* `migrations`
    * `add`--Add a new migration
    * `list`--List the migrations
    * `remove`--Remove the last migration
    * `script`--Generate a SQL script from migrations

Evolving
========
There may be a few rough edges, but please [let us know][4] what you think about this and other Entity Framework Core
features.


  [1]: http://blogs.msdn.com/b/adonet/archive/2014/05/19/ef7-new-platforms-new-data-stores.aspx
  [2]: http://www.asp.net/vnext
  [3]: http://docs.nuget.org/docs/start-here/using-the-package-manager-console
  [4]: https://github.com/aspnet/EntityFramework/issues/new
