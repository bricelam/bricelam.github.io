---
layout: post
title: Entity Framework 7 Migrations in ASP.NET 5
tags: entity-framework
---

One of the [new platforms][1] that we're targeting in Entity Framework 7 is [ASP.NET 5][2]. With this new platform
comes a new set of challenges for how we enable the Migrations commands. Ever since Entity Framework 4.3, we've
provided a set of PowerShell commands that could be run in Visual Studio from [NuGet's Package Manager Console][3].
However, that won't help you if you're developing on OSX where neither PowerShell nor Visual Studio are available. This
post will show you how to use the new Migrations KRE commands.

Installing
==========
In EF7, we're striving for a more modular design. This has resulted in more fine-grained NuGet packages. The new
Migrations commands are contained in the EntityFramework.Commands package. To use it, add it as a dependency to your
project.json file as well as a corresponding command.

{% highlight json %}
{
    "dependencies": {
        "EntityFramework.Commands": "7.0.0-*"
    },
    "commands": {
        "ef": "EntityFramework.Commands"
    }
}
{% endhighlight %}

Once you've done this, you should be able to restore packages, change directory into your project (the directory
containing the project.json file), and run the command.

{% highlight bat %}
kpm restore
cd src\MyProject
k ef
{% endhighlight %}

![Migrations KRE Commands]({{ site.baseurl }}/attachments/EFCommands.png)

Using
=====
To see what sub-commands are available for the `migration` command, type `k ef migration --help`. There are just two for
now, but more will come in the next release.

* `add`--Add a new migration
* `apply`--Apply migrations to the database

To see the usage of the `add` command, type `k ef migration add --help`.

To add a new migration, type `k ef migration add MyMigration`.

Evolving
========
There are still some rough edges, but we wanted to get preview of the new commands out in time for the alpha4 release.
Please [let us know][4] what you think about this and other Entity Framework 7 features.


  [1]: http://blogs.msdn.com/b/adonet/archive/2014/05/19/ef7-new-platforms-new-data-stores.aspx
  [2]: http://www.asp.net/vnext
  [3]: http://docs.nuget.org/docs/start-here/using-the-package-manager-console
  [4]: https://github.com/aspnet/EntityFramework/issues/new
