---
layout: post
title: EF7 NuGet Commands
tags: entity-framework
---

With Entity Framework version 7 comes the great opportunity to improve upon EF's legacy by incorporating all the lessons
we've learned over the past few years of developing this software. For me, that means a chance to improve on one of the
first features I worked on after joining the team over four years ago: Code First Migrations' NuGet Commands. In this
post, I want to talk about some of the improvements to these PowerShell commands.

Boxes and Lines
===============
I want to start by talking about the design. Let's examine the current implementation that has been used since version
4.3 and talk about the motivation behind it.

EF 4.3 Design
-------------
![EF 4.3 NuGet Commands Design]({{ site.baseurl }}/attachments/EF6Commands.png)

Let's walk through it going from left to right. We start with the PowerShell module (`EntityFramework.psm1`) which is
hosted in the main Visual Studio AppDomain. These call into the command classes which are contained in
`EntityFramework.PowerShell.dll`. Originally, this was all done inside the VS domain, but we found that since the
PowerShell assembly was being locked, the `EntityFramework` NuGet package could not be uninstalled or updated. To solve
this, we delayed loading any assemblies until you invoked a command. We also loaded them inside a seperate domain so
that they could be unloaded later. Unfortunately, there were some things that couldn't be done outside of the main VS
domain, and we had to add `EntityFramework.PowerShell.Utility.dll` to perform cross-domain communication. So, we were
still locking an assembly, but it was done lazily. This worked pretty well until EF6 when we started calling PowerShell
commands when the NuGet package was installed. This took us right back to where we started with not being able to
uninstall or update. We needed a better design.

Let's continue walking through the design. The command classes called into the `ToolingFacade` class (contained in
`EntityFramework.dll`) which loaded the user's application into another AppDomain. The purpose of this domain was to
mimic the runtime environment as much as possible so that when we invoked the DbContext and Migrations code, it would
behave as the user expected.

How has all of this been improved in EF7?

EF7 Design
----------
![EF7 NuGet Commands Design]({{ site.baseurl }}/attachments/EF7Commands.png)

We still start in a PowerShell module (`EntityFramework.psm1`). The PowerShell module is now responsible for creating
the runtime-like AppDomain for hosting the user's application. It also takes care of doing things that need to be done
inside the VS domain. We still need cross-domain communication, but instead of loading the same types on both sides of
the boundary like we use to with `EntityFramework.PowerShell.Utility.dll`, We've created a new class called
`ForwardingProxy`. I had to delve pretty deep into the dark world of .NET Remoting (my wound from the Morgul-blade still
aches from time to time), but I found a way of invoking methods on a type that isn't loaded. The target type on the VS
side of the boundary is loaded from source using the `Add-Type` cmdlet. The problem of locking assemblies was finally
solved!

The PowerShell module calls into the `Executor` class which is a thin, cross-domain-friendly wrapper over the
`MigrationTool` class which invokes code inside the user's application (via Reflection).

Shiny River Rocks
=================
In addition to a new design, we've also made some small productivity enhancements to give the experience a nice
polished feel.

Tab Expansion
-------------
When working on projects with multiple DbContext classes, you had to specify which `DbMigrationsConfiguration` class to
use every time you invoked a command. This annoyed @lukewaters greatly. (He's a tester on our team.) To make this
easier, we've enabled tab expansion of DbContext classes and migration names.

![EF7 NuGet Commands Tab Expansion]({{ site.baseurl }}/attachments/CommandsTabExpansion.png)

Use-DbContext
-------------
We went one step farther and added the `Use-DbContext` command. This command allow you to specify a default DbContext to
use for the current PowerShell session. I like to call this the Luke command.

{% highlight powershell %}
Use-DbContext UnicornContext
Add-Migration TwentyPercentCooler
{% endhighlight %}

Less Is More
------------
While playing around with the new commands, you may notice that a few commands and many parameters have been removed.
We've made some design changes to Code First Migrations to enable this, and I'll cover those in a future post. There
is, however one change worth mentioning in this post.

The `Update-Database` command has been split into two new commands.

* `Apply-Migration` applies migrations to the database
* `Script-Migration` generates a SQL script to apply the migrations to the database

New Platforms
-------------
EF7 is being built for new platforms including Windows 8.1 and Windows Phone 8.1. I'm happy to say that, in addition to
the Windows Desktop projects, the commands will also work with Windows, Windows Phone, and Portable projects.

Feedback
========
As always, please let us know what you think about this and other Entity Framework 7 features by commenting on [our
blog posts][1], [tweeting us][2], or [submitting new issues][3].


  [1]: http://blogs.msdn.com/adonet/
  [2]: https://twitter.com/efmagicunicorns
  [3]: https://github.com/aspnet/EntityFramework/issues/new
