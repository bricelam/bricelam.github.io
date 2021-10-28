---
layout: post
title: A History of Entity Framework
tags: entity-framework
---

I recently had the honor of appearing on [The 6 Figure Developer](https://6figuredev.com/) podcast. I reminisced about the history of Entity Framework and the role I've been blessed to play in it. Have a listen!

* [Episode 132 – EF and EF Core with Brice Lambson](https://6figuredev.com/podcast/episode-132-ef-an-ef-core-with-brice-lambson/)

I put together a timeline to go with the episode:

Year | Event
---- | -----
2006 | [WinFS cancelled](https://docs.microsoft.com/archive/blogs/winfs/winfs-update)<br /><br />[First preview](https://docs.microsoft.com/archive/blogs/dataaccess/ado-net-vnext-the-entity-framework-linq-and-more) of Entity Framework released
2008 | [Vote of no confidence](https://www.zdnet.com/article/testers-give-microsofts-entity-framework-a-no-confidence-vote/)<br /><br />[EF 1.0](https://docs.microsoft.com/archive/blogs/adonet/rtm-is-finally-here) released as part of .NET Framework 3.5 SP1<br /><br />[LINQ to SQL cancelled](https://docs.microsoft.com/archive/blogs/adonet/update-on-linq-to-sql-and-linq-to-entities-roadmap)
2009 | [Programming Entity Framework](http://shop.oreilly.com/product/9780596807252.do) (by Julie Lerman) published
2010 | [EF 4.0](https://docs.microsoft.com/archive/blogs/adonet/entity-framework-and-linq-to-sql-additional-programming-patterns-in-ef4) released (POCO, foreign keys, lazy loading, templates)<br /><br />[Magic Unicorn Edition](https://www.hanselman.com/blog/SimpleCodeFirstWithEntityFramework4MagicUnicornFeatureCTP4.aspx) released<br /><br />I joined the team<br /><br />[Programming Entity Framework: Code First](http://shop.oreilly.com/product/0636920022220.do) (by Julie Lerman and Rowan Miller) published
2011 | [Programming Entity Framework: DbContext](http://shop.oreilly.com/product/0636920022237.do) (by Julie Lerman and Rowan Miller) published<br /><br />[EF 4.1](https://docs.microsoft.com/archive/blogs/adonet/ef-4-1-released) released (DbContext, code first)
2012 | [EF 4.3](https://docs.microsoft.com/archive/blogs/adonet/ef-4-3-released) released (migrations)<br /><br />Went [open source](https://docs.microsoft.com/archive/blogs/adonet/entity-framework-and-open-source)<br /><br />[EF 5.0](https://docs.microsoft.com/archive/blogs/adonet/ef5-released) released (enums, spatial, TVFs)<br /><br />Move from SQL org to Azure (ASP.NET)
2013 | [EF 6.0](https://docs.microsoft.com/archive/blogs/adonet/ef6-rtm-available) released (async, connection resiliency, custom conventions, sprocs)
2014 | [EF 6.1](https://docs.microsoft.com/archive/blogs/adonet/ef6-1-0-rtm-available) released<br /><br />Began work on [EF7](https://docs.microsoft.com/archive/blogs/adonet/ef7-new-platforms-new-data-stores)
2016 | EF7 [renamed to EF Core](https://www.hanselman.com/blog/ASPNET5IsDeadIntroducingASPNETCore10AndNETCore10.aspx)<br /><br />[EF Core 1.0](https://devblogs.microsoft.com/dotnet/entity-framework-core-1-0-0-available/) released (mixed-eval, shadow state properties, unique constraints, sequences, batching, attach graph APIs)<br /><br />[EF Core 1.1](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-1-1/) released (memory-optimized tables)
2017 | [EF Core 2.0](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-2-0/) released (table splitting, owned entities, global query filters, function mapping)<br /><br />[EF 6.2](https://devblogs.microsoft.com/dotnet/entity-framework-6-2-runtime-released/) released
2018 | [EF Core 2.1](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-2-1/) released (lazy loading, value converters, keyless entity types, group by, constructor parameters, seeding)<br /><br />[Entity Framework Core in Action](https://www.manning.com/books/entity-framework-core-in-action) (by Jon P Smith) published<br /><br />[EF Core 2.2](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-2-2/) released (spatial, owned collections, query tags)
2019 | Moved to DevDiv (.NET) org<br /><br />[EF Core 3.0](https://devblogs.microsoft.com/dotnet/announcing-ef-core-3-0-and-ef-6-3-general-availability/) released (Azure Cosmos DB, await foreach, nullable reference types, single server-eval query, interceptors)<br /><br />[EF 6.3](https://devblogs.microsoft.com/dotnet/announcing-ef-core-3-0-and-ef-6-3-general-availability/#what-s-new-in-ef-6-3) released (.NET Core)<br /><br />[EF 6.4](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-3-1-and-entity-framework-6-4/) released<br /><br />[EF Core 3.1](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-3-1-and-entity-framework-6-4/) released
2020 | [Entity Framework Community Standup](https://youtube.com/playlist?list=PLdo4fOcmZ0oX0ObHwBrJ0vJpZ7PiYMqeA) premiered<br /><br />[EF Core 5.0](https://devblogs.microsoft.com/dotnet/announcing-the-release-of-ef-core-5-0/) released (many-to-many, TPT, collations, TVFs, filtered include, property bags, table rebuilds, exclude from migrations, change-tracking proxies)
2021 | [EF Core 6.0](https://docs.microsoft.com/ef/core/what-is-new/ef-core-6.0/plan) planned (perf improvements, compiled models, migration bundles, temporal tables)

If you're interested in learning more about the history of EF, here are some very interesting blogs you can dig through:

* [WinFS Team Blog](https://docs.microsoft.com/archive/blogs/winfs/)
* [Data Access Blog](https://docs.microsoft.com/archive/blogs/dataaccess/)
* [ADO.NET Blog](https://docs.microsoft.com/archive/blogs/adonet/)
* [EF Design Blog](https://docs.microsoft.com/archive/blogs/efdesign/)
