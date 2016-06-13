---
layout: post
title: 'EF7 Migrations: Runtime'
tags: entity-framework
---

This post is part of my series on EF7 Migrations. In previous posts, I've talked about the [DNX][1] & [NuGet][2] commands and the
[design-time][3] experience of Migrations. In this post, I'll talk about the runtime experience (or API) of Migrations.

Building Migrations
===================
TODO:

* MigrationBuilder

Extending
---------
{% highlight csharp %}
public static void CreateSynonym(
    this MigrationBuilder migration,
    string name,
    string forObject)
{
    migration.Add(
        new SqlOperation
        {
            Sql = $"CREATE SYNONYM {name} FOR {forObject};";
        });
}
{% endhighlight %}

Model Snapshots
===============
TODO:

* Only when merging
* Like OnModelCreating without conventions

Runtime Migrations
==================
TODO:

* Mostly mobile

{% highlight csharp %}
options.UseSqlServer().MigrationsAssembly("App1Migrations");
db.Database.Relational().ApplyMigrations();
{% endhighlight %}


  [1]: {% post_url 2014-09-14-migrations-on-k %}
  [2]: {% post_url 2014-10-22-ef7-nuget-commands %}
  [3]: {% post_url 2014-12-16-ef7-migrations-designtime %}
