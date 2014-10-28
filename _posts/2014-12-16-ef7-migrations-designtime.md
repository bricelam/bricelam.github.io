---
layout: post
title: 'EF7 Migrations: Design-time'
tags: entity-framework
---

In previous posts, I've talked the [new ASP.NET commands][1] and the [enhanced NuGet commands][2] in Entity Framework 7.
This post dives into some of the changes we've made to enhance the design-time experience of Migrations.

Enabling Migrations
===================
We've updated `Enable-Migrations` to output the following message.

> The term 'Enable-Migrations' is not recognized as the name of a cmdlet, function, script file, or operable program.
> Check the spelling of the name, or if a path was included, verify that the path is correct and try again.

Just kidding. We've removed it. :wink:

`DbMigrationsConfiguration` has been removed; thus, there's no need for `Enable-Migrations`. To start using Migrations,
just type `Add-Migration`.

Automatic Migrations
====================
Automatic migrations were awesome ...for demos. We've removed them in EF7, and I have an appointment scheduled with Dr.
Howard Mierzwiak to remove them from my memory.

Automatic migrations worked by diffing the last-known model against the current model. The last-known model may be
stored in either the last explicit migration (i.e. non-automatic) or in the database if it comes from another automatic
migration. Because the last migration may not be an explicit migration, we always had to look in the database. Since we
always looked in the database, we wouldn't allow you to scaffold multiple migrations without first applying them.
Automatic migrations also required us to store both the source and target models for each explicit migration that
succeeded an automatic migration. Are you starting to see how these may have hindered the design-time experience?

By removing automatic migrations in EF7, we're able to...

* Stop storing model snapshots in the database
* Stop querying the database on `Add-Migration`
* Allow `Add-Migration` without applying the last migration
* Stop storing source model snapshots in the migrations

To see just how awesome all of this is, try the following.

{% highlight powershell %}
Add-Migration M1
# (Tweak the model)
Add-Migration M2 # Would fail on EF6
# (Tweak the model)
Add-Migration M3
Apply-Migration # Applies M1, M2 & M3
{% endhighlight %}

Model Snapshots
===============
Model snapshots are awesome. The allow us to diff against your current model and scaffold the operations to get your
database up to date. They did, however, cause a lot of problems in one particular area: team environments. We have an
entire article, [Code First Migrations in Team Environments][3], that describes these problems and how to avoid them.

The heart of the problem is that we stored the model snapshot inside each individual migration. By doing this, we lost
all of the automatic merging and conflict detection that source control management provides.

To fix this, we've moved the model snapshot into its own file. Now, when you pull in another migration that someone else
authored, you will get merging and conflicts from source control just like any other file. If you do get a legitimate
conflict, however, you'll probably need to manually update one of the migration files. For example, if they renamed
column A to B, and you renamed column A to C, update yours to rename column B instead.

As full disclosure, we actually do still store a model snapshot in each migration, but it's not used for diffing. It's
there in case you or a provider need to inspect the model for additional information during a migration.

Remove-Migration
----------------
One consequence of moving the model snapshot is that you can no longer simply delete migration files. The model snapshot
also has to be reverted. Otherwise, your next `Add-Migration` will diff against a model that is incorrect. To help do
this, we've added the `Remove-Migration` command. It will delete the last migration's files and revert the model
snapshot. It's also smart enough to detect when the migration's files have been manually deleted and, if they have, just
revert the model snapshot.

It's Your Code
==============
We've also done a few things to let you have greater control over your code.

Migrations can now span multiple namespaces. For example, if you want to group them by, say, what release cycle they
were created for you could have some in `MyProject.Migration.V1_0` and some in `MyProject.Migration.V1_1`.
`Add-Migration` is even smart enough to create new migrations in the same namespace and folder as the last one.

Like in EF6, you can put migrations in a separate assembly. Though in EF7, we removed some of the pain. Just remember to
always invoke Migrations commands on the project containing the migrations.

Feedback
========
As always, please let us know what you think about this and other Entity Framework 7 features by commenting on [our
blog posts][4], [tweeting us][5], or [submitting new issues][6].


  [1]: {% post_url 2014-09-14-migrations-on-k %}
  [2]: {% post_url 2014-10-22-ef7-nuget-commands %}
  [3]: http://msdn.microsoft.com/en-us/data/dn481501
  [4]: http://blogs.msdn.com/adonet/
  [5]: https://twitter.com/efmagicunicorns
  [6]: https://github.com/aspnet/EntityFramework/issues/new
