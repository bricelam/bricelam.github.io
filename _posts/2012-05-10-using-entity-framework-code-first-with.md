---
layout: post
title: Using Entity Framework 4.3 Code First with MySQL's Connector/Net
permalink: /2012/05/using-entity-framework-code-first-with.html
tags: [ entity-framework, mysql ]
---

<div style="color: red; text-align: center;">
  <hr />
  <span style="font-size: large;">This issue has been resolved.</span><br />
  For versions 6.6.3 or newer of Connector/Net, this workaround is not required.
  <hr />
</div>

Our team recently came across [a Chinese post][1] reporting an issue when using Code First with MySQL. You get the
following exception while trying to create the database.

    MySql.Data.MySqlClient.MySqlException: You have an error in
    your SQL syntax; check the manual that corresponds to your
    MySQL server version for the right syntax to use near 'NOT
    NULL,
            `ProductVersion` mediumtext NOT NULL);

    ALTER TABLE `__MigrationH' at line 5

The Problem
===========
If you look at the full SQL that it is trying to run, the problem becomes clearer.

```sql
CREATE TABLE `__MigrationHistory` (
    `MigrationId` mediumtext NOT NULL,
    `Model` varbinary NOT NULL,
    `ProductVersion` mediumtext NOT NULL);

ALTER TABLE `__MigrationHistory`
ADD PRIMARY KEY (MigrationId);
```

In MySQL, `varbinary` types must specify a max length. That's not the only problem though; a `mediumtext` primary key
also must specify a key length. Interestingly, if you look at your database after recieving this exception, all of your
tables are created and, if you try to run your app again everything appears to work. So what's the problem? The problem
is that there is no `__MigrationHistory` table. This table is essential for the [Database.CompatibleWithModel][2] method
to work properly which, in turn, is used by the [CreateDatabaseIfNotExists][3] and [DropCreateDatabaseIfModelChanges][4]
database initializers.

A Workaround
============
Until the [Connector/Net][5] provider is updated to properly handle the `__MigrationHistory` table, we'll need to create
it ourselves fixing the two problems mentioned above. I've created a database initializer to do this for you modeled
after the behavior of the `CreateDatabaseIfNotExists` initializer. Most of the code here can also be used to create one
that mirrors `DropCreateDatabaseIfModelChanges` too. Here it is.

```cs
class CreateMySqlDatabaseIfNotExists<TContext>
    : IDatabaseInitializer<TContext>
        where TContext : DbContext
{
    public void InitializeDatabase(TContext context)
    {
        if (context.Database.Exists())
        {
            if (!context.Database.CompatibleWithModel(false))
            {
                throw new InvalidOperationException(
                    "The model has changed!");
            }
        }
        else
        {
            CreateMySqlDatabase(context);
        }
    }

    private void CreateMySqlDatabase(TContext context)
    {
        try
        {
            // Create as much of the database as we can
            context.Database.Create();

            // No exception? Don't need a workaround
            return;
        }
        catch (MySqlException ex)
        {
            // Ignore the parse exception
            if (ex.Number != 1064)
            {
                throw;
            }
        }

        // Manually create the metadata table
        using (var connection = ((MySqlConnection)context
            .Database.Connection).Clone())
        using (var command = connection.CreateCommand())
        {
            command.CommandText =
@"
CREATE TABLE __MigrationHistory (
    MigrationId mediumtext NOT NULL,
    Model mediumblob NOT NULL,
    ProductVersion mediumtext NOT NULL);

ALTER TABLE __MigrationHistory
ADD PRIMARY KEY (MigrationId(255));

INSERT INTO __MigrationHistory (
    MigrationId,
    Model,
    ProductVersion)
VALUES (
    'InitialCreate',
    @Model,
    @ProductVersion);
";
            command.Parameters.AddWithValue(
                "@Model",
                GetModel(context));
            command.Parameters.AddWithValue(
                "@ProductVersion",
                GetProductVersion());

            connection.Open();
            command.ExecuteNonQuery();
        }
    }

    private byte[] GetModel(TContext context)
    {
        using (var memoryStream = new MemoryStream())
        {
            using (var gzipStream = new GZipStream(
                memoryStream,
                CompressionMode.Compress))
            using (var xmlWriter = XmlWriter.Create(
                gzipStream,
                new XmlWriterSettings { Indent = true }))
            {
                EdmxWriter.WriteEdmx(context, xmlWriter);
            }

            return memoryStream.ToArray();
        }
    }

    private string GetProductVersion()
    {
        return typeof(DbContext).Assembly
            .GetCustomAttributes(false)
            .OfType<AssemblyInformationalVersionAttribute>()
            .Single()
            .InformationalVersion;
    }
}
```

There you have it. We basically let the Database.Create call do as much work as it can, then take over when it fails to
create the `__MigrationHistory` table.

You can use the new initializer by calling `Database.SetInitializer`. One of the best places to do this is in your
context's static constructor.

```cs
class MyContext : DbContext
{
    static MyContext()
    {
        Database.SetInitializer(
            new CreateMySqlDatabaseIfNotExists<MyContext>();
    }

    public MyContext()
        : base("Name=LocalMySqlServer")
    {
    }

    // Add DbSet properties here
}
```

Alternatively, you can set it in your App/Web.config.

```xml
<entityFramework>
  <contexts>
    <context type="MyNamespace.MyContext, MyAssembly">
      <databaseInitializer type="MyNamespace.
CreateMySqlDatabaseIfNotExists`1[[MyNamespace.MyContext,
MyAssembly]], MyAssembly" />
    </context>
  </contexts>
</entityFramework>
```

The Fix
=======
Like any good open source software user, I've filed two bugs with the Connecter/Net team. You can check the status to
see what progress has been made towards an actual fix for the problem.

* [Bug #65289][6] - Cannot create an entity with a key of type string
* [Bug #65290][7] - Cannot create an entity with a property of type byte[]


  [1]: http://www.microsofttranslator.com/bv.aspx?from=zh-CHS&amp;to=en&amp;a=http://www.cnblogs.com/shouzheng/archive/2012/03/09/2388177.html
  [2]: http://msdn.microsoft.com/en-us/library/system.data.entity.database.compatiblewithmodel.aspx
  [3]: http://msdn.microsoft.com/en-us/library/gg679221.aspx
  [4]: http://msdn.microsoft.com/en-us/library/gg679604.aspx
  [5]: http://dev.mysql.com/downloads/connector/net
  [6]: http://bugs.mysql.com/bug.php?id=65289
  [7]: http://bugs.mysql.com/bug.php?id=65290
