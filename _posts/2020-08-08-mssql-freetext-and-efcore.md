---
layout: post
title: SQL Server Full-Text Search and EF Core
tags: [ entity-framework, sql-server ]
---

[Full-Text Search](https://docs.microsoft.com/sql/relational-databases/search/full-text-search) is a feature of Microsoft SQL Server that lets you perform search engine like queries against the string properties of your entities.

The Full-Text feature isn't available on the LocalDB (the version of SQL Server that comes with Visual Studio). I like to use Developer Edition on Docker for local development.

``` sh
docker run -d -p 1433:1433 -e SA_PASSWORD=Password12! -e ACCEPT_EULA=Y mcr.microsoft.com/mssql/server
```

Use this connection string to connect to the docker image.

    Data Source=(local)\MSSQLSERVER;Initial Catalog=BlogsDatabase;User ID=sa;Password=Password12!

Now that we've got the prerequisites installed, let's assume we're starting with a Post entity type that has a Content property we'd like to search.

``` cs
class Post
{
    public int Id { get; set; }
    public string Content { get; set; }
}
```

First, we need to add a full-text index to the Content column. Do this by adding a new Migration.

``` sh
dotnet ef migrations add AddFullTextIndexToPostContent
```

Inside the `Up` method of the migration, add the following.

``` cs
migrationBuilder.Sql(
    sql: "CREATE FULLTEXT CATALOG ftCatalog AS DEFAULT;",
    suppressTransaction: true);

migrationBuilder.Sql(
    sql: "CREATE FULLTEXT INDEX ON Posts(Content) KEY INDEX PK_Posts;",
    suppressTransaction: true);
```

The first operation adds a full-text catalog. This serves as a container for any full-text indexes.

The second operation creates the full-text index enabling queries on the column.

Neither operation can be executed inside a transaction, so we need to suppress the migration's transaction. Warning, this currently doesn't work when applying migrations during publish from Visual Studio (see [dotnet/sdk#12676](https://github.com/dotnet/sdk/issues/12676)).

Don't forget to apply the migration.

``` sh
dotnet ef database update
```

To issue a full-text query from EF, use the Contains or FreeText functions:

``` cs
var results = from p in db.Posts
              where EF.Functions.FreeText(p.Content, query)
              select p;
```

The query string has an entire syntax of its own. Check out the [Full-Text Query](https://docs.microsoft.com/sql/relational-databases/search/query-with-full-text-search) documentation to learn more.

Happy searching!
