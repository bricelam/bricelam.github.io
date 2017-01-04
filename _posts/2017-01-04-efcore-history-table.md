---
layout: post
title: Custom Migrations History Table in EF Core
tags: entity-framework
---

This post demonstrates the different ways you can customize the migrations history table in EF Core.

The simplest scenario is when you just want to change the table name or schema. This can be done using the
`MigrationsHistoryTable` method in `OnConfiguring` (or `ConfigureServices` on ASP.NET Core). Here is an example.

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
    => options.UseSqlServer(
        connectionString,
        x => x.MigrationsHistoryTable("__MyMigrationsHistory", "mySchema"));
```

You can also customize the columns, but this is a bit more involved. You need to override and replace the
provider-specific `IHistoryRepository` service. If the provider uses the base `HistoryRepository` implementation, you
can customize the columns by overriding the `ConfigureTable` method. Here is an example of changing the MigrationId
column name to *Id*.

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
    => options
        .UseSqlServer(connectionString)
        .ReplaceService<SqlServerHistoryRepository, MyHistoryRepository>();
```

```csharp
class MyHistoryRepository : SqlServerHistoryRepository
{
    public MyHistoryRepository(
        IDatabaseCreator databaseCreator, IRawSqlCommandBuilder rawSqlCommandBuilder,
        ISqlServerConnection connection, IDbContextOptions options,
        IMigrationsModelDiffer modelDiffer,
        IMigrationsSqlGenerator migrationsSqlGenerator,
        IRelationalAnnotationProvider annotations,
        ISqlGenerationHelper sqlGenerationHelper)
        : base(databaseCreator, rawSqlCommandBuilder, connection, options,
            modelDiffer, migrationsSqlGenerator, annotations, sqlGenerationHelper)
    {
    }

    protected override void ConfigureTable(EntityTypeBuilder<HistoryRow> history)
    {
        base.ConfigureTable(history);

        history.Property(h => h.MigrationId).HasColumnName("Id");
    }
}
```
