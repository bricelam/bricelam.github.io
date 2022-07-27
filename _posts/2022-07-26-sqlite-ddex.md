---
layout: post
title: SQLite in Visual Studio 2022
tags: sqlite
---

A couple of years ago, I was thinking of ways to see if any fundamental ADO.NET features were still missing from [Microsoft.Data.Sqlite](https://docs.microsoft.com/en-us/dotnet/standard/data/sqlite/?tabs=netcore-cli), or if it broke any long-established assumptions. I decided to try implementing a Visual Studio [Data Designer Extensibility](https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2013/bb165128(v=vs.120)) (DDEX) provider.

This technology is very old, and I suspect that parts of it even existed before .NET. Seeing an ADO.NET provider from DDEX's perspective gave me a lot of insight into the design of ADO.NET. For example, the [GetSchema method and its collections](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/common-schema-collections) were always strange to me, and frankly, seemed kinda useless. But now, I see they exist primarily to support the DDEX provider. My new opinion is that GetSchema is actually just the result of bad architectural layering. 😉

I've been steadily making progress on this provider in my spare time, and I've found that having a read-only view of SQLite databases inside of Visual Studio's Server Explorer can be pretty handy when debugging. It'll never be able to compete with more robust tools like [SQLite Toolbox](https://marketplace.visualstudio.com/items?itemName=ErikEJ.SQLServerCompactSQLiteToolbox), [DB Browser for SQLite](https://sqlitebrowser.org/), or [DataGrip](https://www.jetbrains.com/datagrip/), but coupled with the fact that it's also a DDEX provider for Microsoft.Data.Sqlite that other Visual Studio extensions could use, I decided to release a preview.

You can download the preview from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=bricelam.VSDataSqlite) or from the Manage Extensions dialog inside Visual Studio 2022. I'm eager to see if you think it's useful.

![VisualStudio.Data.Sqlite screenshot]({{ "/attachments/sqlite-ddex.png" | absolute_url }})
