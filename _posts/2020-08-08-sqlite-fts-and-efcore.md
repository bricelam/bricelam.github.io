---
layout: post
title: SQLite Full-Text Search and EF Core
tags: [ entity-framework, sqlite ]
date: 2020-09-08
---

[FTS5](https://www.sqlite.org/fts5.html) is an extension to SQLite that enables search engine like queries against string properties in your entities.

Let's assume we're starting with a Post entity type that has a Content property we'd like to search.

``` cs
class Post
{
    public int Id { get; set; }
    public string Content { get; set; }
}
```

FTS5 requires you to create an entirely new table. I prefer leaving the existing table mostly intact and only moving the text columns that I want to search into this new table. You can link the two table with a one-to-one relationship. Note, a more natural mapping with only one entity type can be enabled by the [entity splitting](https://github.com/dotnet/efcore/issues/620) feature.

Here's a new entity type to represent the FTS5 table we're going to add.

``` cs
class FTSPost
{
    public int RowId { get; set; }
    public Post Post { get; set; }

    public string Content { get; set; }

    public string Match { get; set; }
    public double? Rank { get; set; }
}
```

RowId is a hidden column in SQLite used to uniquely identity each row in a table. We'll use this as our entity type key and as a foreign key to Post.

Match and Rank are used to represent hidden FTS5 columns that we'll use when querying. The Match property needs to map to a column with the same name as the table.

``` cs
modelBuilder.Entity<FTSPost>(x =>
{
    x.HasKey(fts => fts.RowId);

    x.Property(fts => fts.Match).HasColumnName(nameof(FTSPost));

    x.HasOne(fts => fts.Post).WithOne(p => p.FTS)
        .HasForeignKey<FTSPost>(fts => fts.RowId);
});
```

Don't forget to remove the Content property from Post and add a navigation property.

``` cs
class Post
{
    public int Id { get; set; }

    // Moved to FTSPost
    //public string Content { get; set; }

    public FTSPost FTS { get; set; }
}
```

Now that we have the EF model the way we want it, let's add a new [migration](https://docs.microsoft.com/ef/core/managing-schemas/migrations/) and use it to create the table. We'll rearrange the order of operations a bit so we can copy data from the old table into the new one. Warning, the `DropColumn` operation will fail on versions of EF Core older than 5.0, but since the column is nullable, you can just remove the operation from the migration and let your app ignore the old column.

``` cs
// UNDONE: Using an FTS5 table instead
//migrationBuilder.CreateTable(
//    name: "FTSPost",
//    columns: table => new
//    {
//        Content = table.Column<string>()
//    });
migrationBuilder.Sql(@"
    CREATE VIRTUAL TABLE FTSPost USING fts5(Content);

    INSERT INTO FTSPost (rowid, Content)
    SELECT Id, Content
    FROM Posts;
");

// TODO: Just comment this out on EF Core 3.1
migrationBuilder.DropColumn(
    name: "Content",
    table: "Posts");
```

Our database is ready now. We're ready to query. To help us, let's map the Highlight and Snippet FTS5 functions using the `[DbFunction]` attribute on some methods in our DbContext.

``` cs
[DbFunction]
public string Highlight(string match, string column, string open, string close)
    => throw new NotImplementedException();

[DbFunction]
public string Snippet(string match, string column, string open, string close, string ellips, int count)
    => throw new NotImplementedException();
```

Here is an example showing how to query.

``` cs
var results = from fts in db.Set<FTSPost>()
              where fts.Match == query
              orderby fts.Rank
              select new
              {
                  PostId = fts.RowId,
                  Snippet = db.Snippet(fts.Match, fts.Content, "<b>", "</b>", "...", 32)
              };
```

We're abusing the equals operator a bit since there is currently no way to generate a MATCH expression in the query. [In the future](https://github.com/dotnet/efcore/issues/4823), we should enable something like `EF.Functions.Match(fts, query)` instead.

The query string has an entire syntax of it own. Be sure to check out the [FTS5](https://www.sqlite.org/fts5.html#full_text_query_syntax) documentation for details.

The hidden Rank column is used to order the results by most relevant.

The Snippet function returns a string roughly 32 words in length with the matching words highlighted. The Highlight function is similar, but returns the entire value of the column.

I was careful to avoid a JOIN to the Posts table in my query since there seems to be some bugs in FTS5 that cause no results to be returned when you do this. Be sure to test your queries.

Happy searching! There are a lot of rough edges here, so let me know if you find a more elegant solution. Also, [let us know](https://github.com/dotnet/efcore/issues/4823) if you can think of any additional enhancements we can make to EF that makes this experience better.
