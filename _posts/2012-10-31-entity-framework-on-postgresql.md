---
layout: post
title: Entity Framework 5 on PostgreSQL
permalink: /2012/10/entity-framework-on-postgresql.html
tags: entity-framework postgresql
---

This walkthrough will get you started with an application that uses the [Entity Framework][1] (EF) to read and write
data from a [PostgreSQL][2] database. It is intended to be similar to the [Code First to a New Database][3] walkthrough.

There are currently two PostgreSQL providers for EF that I know of: [Npgsql][4] and [Devart's dotConnect for
PostgreSQL][5]. Devart's provider has a much richer set of features, but it is also a commercial product. In the spirit
of [FOSS][6], we will be using the Npgsql provider for this walkthrough. I encourage you to keep the Devart provider in
mind, however, if your project requires that extra level of support.

Create the Application
======================
For simplicity, we will be using a Console Application, but the basic steps are the same regardless of project type.

1. Open Visual Studio
2. Select **File** -> **New** -> **Project...**
3. Select **Console Application**
4. Name the project
5. Click **OK**

Create the Model
================
For our model, we'll be borrowing pieces from the [Chinook Database][7] (a cross-platform, sample database).
Specifically, we will be using Artists and Albums.

Add the following two classes to your project.

{% highlight csharp %}
public class Artist
{
    public Artist()
    {
        Albums = new List<Album>();
    }

    public int ArtistId { get; set; }
    public string Name { get; set; }

    public virtual ICollection<Album> Albums { get; set; }
}

public class Album
{
    public int AlbumId { get; set; }
    public string Title { get; set; }

    public int ArtistId { get; set; }
    public virtual Artist Artist { get; set; }
}
{% endhighlight %}

Create a Context
================
In EF, the context becomes your main entry point into the database. Before we define our context though, we will need to
install Entity Framework.

1. Select **Tools** -> **Library Package Manager** -> **Package Manager Console**
2. Inside the Package Manager Console (PMC) run `Install-Package EntityFramework`

Now, add the context class to your project.

{% highlight csharp %}
class ChinookContext : DbContext
{
    public DbSet<Artist> Artists { get; set; }
    public DbSet<Album> Albums { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Map to the correct Chinook Database tables
        modelBuilder.Entity<Artist>().ToTable("Artist", "public");
        modelBuilder.Entity<Album>().ToTable("Album", "public");

        // Chinook Database for PostgreSQL doesn't auto-increment Ids
        modelBuilder.Conventions
            .Remove<StoreGeneratedIdentityKeyConvention>();
    }
}
{% endhighlight %}

Install the Provider
====================
In order to connect to PostgreSQL databases, we will need to install an appropriate ADO.NET and Entity Framework
provider. Luckily, the provider we're using is available via NuGet.

1. Inside PMC, run `Install-Package Npgsql`

We also need to register the provider. Open App.config, and anywhere inside the configuration element, add the following
fragment.

{% highlight xml %}
<system.data>
  <DbProviderFactories>
    <add name="Npgsql Data Provider"
          invariant="Npgsql"
          description="Data Provider for PostgreSQL"
          type="Npgsql.NpgsqlFactory, Npgsql" />
  </DbProviderFactories>
</system.data>
{% endhighlight %}

Add the Database
================
Unfortunately, Npgsql does not support creating databases. So instead of letting Code First create our database, we will
need to manually add a database to our project. It's a good thing we're using a cross-platform sample database!

When I started writing this, Chinook Database actually wasn't available for PostgreSQL. Fortunately, it's an open source
project, so I've submitted a pull request that adds support for PostgreSQL. It was released as part of version 1.4.

1. Download and extract the PostgreSQL version of the [Chinook Database][8]
2. Run **CreatePostgreSql.bat**

Also, add a connection string to the App.Config that points to the database. Anywhere inside the `configuration`
element, add the following fragment.

{% highlight xml %}
<connectionStrings>
  <add name="ChinookContext"
        connectionString=
"Server=localhost;Database=chinook;User Id=postgres;Password=P4ssw0rd;"
        providerName="Npgsql" />
</connectionStrings>
{% endhighlight %}

Start Coding
============
Ok, we should be ready to start coding our application. Let's see what artists exist in the database. Inside Program.cs,
add the following to `Main`.

{% highlight csharp %}
using (var db = new ChinookContext())
{
    var artists = from a in db.Artists
                  where a.Name.StartsWith("A")
                  orderby a.Name
                  select a;

    foreach (var artist in artists)
    {
        Console.WriteLine(artist.Name);
    }
}
{% endhighlight %}

Hmm, it looks like one of my favorite bands is missing. Let's add it.

{% highlight csharp %}
using (var db= new ChinookContext())
{
    db.Artists.Add(
        new Artist
        {
            ArtistId = 276,
            Name = "Anberlin",
            Albums =
            {
                new Album
                {
                    AlbumId = 348,
                    Title = "Cities"
                },
                new Album
                {
                    AlbumId = 349,
                    Title = "New Surrender"
                }
            }
        });
    db.SaveChanges();
}
{% endhighlight %}

We can also update and delete existing data like this.

{% highlight csharp %}
using (var context = new ChinookContext())
{
    var police = db.Artists.Single(a => a.Name == "The Police");
    police.Name = "Police, The";

    var avril = db.Artists.Single(a => a.Name == "Avril Lavigne");
    context.Artists.Remove(avril);

    db.SaveChanges();
}
{% endhighlight %}

Conclusion
==========
Hopefully by now, you have enough information to get started using the Entity Framework with a PostgreSQL database. For
many, many more articles on how to use EF, check out our team's official [Getting Started][9] page on MSDN.


  [1]: http://msdn.com/data/ef
  [2]: http://www.postgresql.org
  [3]: http://msdn.microsoft.com/en-us/data/jj193542
  [4]: http://npgsql.projects.postgresql.org
  [5]: http://www.devart.com/dotconnect/postgresql
  [6]: http://en.wikipedia.org/wiki/Free_and_open-source_software
  [7]: http://chinookdatabase.codeplex.com
  [8]: http://chinookdatabase.codeplex.com/releases
  [9]: http://msdn.microsoft.com/en-us/data/ee712907
