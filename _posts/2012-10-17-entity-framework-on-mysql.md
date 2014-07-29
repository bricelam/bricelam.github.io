---
layout: post
title: Entity Framework 4.2 on MySQL
permalink: /2012/10/entity-framework-on-mysql.html
tags: entity-framework mysql
---

This walkthrough will get you started with an application that uses the [Entity Framework][1] (EF) to read and write
data from a [MySQL][2] database. It is intended to be similar to the [Code First to a New Database][3] walkthrough.

There are currently two MySQL providers for EF that I know of: [Connector/Net][4] and [Devart's dotConnect for
MySQL][5]. Devart's provider has a few more features, but it is also a commercial product. In the spirit of [FOSS][6],
we will be using the Connector/Net provider for this walkthrough. I encourage you to keep the Devart provider in mind,
however, if your project requires that extra level of support.

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

    public int Id { get; set; }
    public string Name { get; set; }

    public virtual ICollection<Album> Albums { get; set; }
}

public class Album
{
    public int Id { get; set; }
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
2. Inside the Package Manager Console (PMC) run `Install-Package EntityFramework -Version 4.2`

> **Note:** We're using EF 4.2 because there's a bug in the current version of Connecter/Net that prevents it from
> properly creating the database. Please see [my workaround][8] if you're interested in using the latest version of EF.

Now, add the context class to your project.

{% highlight csharp %}
class ChinookContext : DbContext
{
    public DbSet<Artist> Artists { get; set; }
    public DbSet<Album> Albums { get; set; }
}
{% endhighlight %}

Install the Provider
====================
In order to connect to MySQL databases, we will need to install an appropriate ADO.NET and Entity Framework provider.
Luckily, the provider we're using is available via NuGet.

1. Inside PMC, run `Install-Package MySQL.Data.Entities`

We also need to register the provider. Open App.config, and anywhere inside the `configuration` element, add the
following fragment.

{% highlight xml %}
<system.data>
  <DbProviderFactories>
    <add name="MySQL Data Provider"
         invariant="MySql.Data.MySqlClient"
         description="Data Provider for MySQL"
         type="MySql.Data.MySqlClient.MySqlClientFactory, MySql.Data" />
  </DbProviderFactories>
</system.data>
{% endhighlight %}

Add a Connection String
=======================
In order to connect to the right database, we need to add a connection string to the App.Config. Anywhere inside the
`configuration` element, add the following fragment.

{% highlight xml %}
<connectionStrings>
  <add name="ChinookContext"
        connectionString=
"server=localhost;database=Chinook;User Id=root;password=P4ssw0rd"
        providerName="MySql.Data.MySqlClient" />
</connectionStrings>
{% endhighlight %}

Start Coding
============
Ok, we should be ready to start coding our application. Let's create some artists and albums.

{% highlight csharp %}
using (var db = new ChinookContext())
{
    db.Artists.Add(
        new Artist
        {
            Name = "Anberlin",
            Albums =
            {
                new Album { Title = "Cities" },
                new Album { Title = "New Surrender" }
            }
        });
    db.Artists.Add(
        new Artist
        {
            Name = "The Police",
            Albums =
            {
                new Album { Title = "The Police Greatest Hits" }
            }
        });
    db.Artists.Add(new Artist { Name = "Avril Lavigne" });
    db.SaveChanges();
}
{% endhighlight %}

Now let's see how to read the data from the database.

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

Finally, here is some code that updates and deletes some data.

{% highlight csharp %}
using (var db = new ChinookContext())
{
    var police = db.Artists.Single(a => a.Name == "The Police");
    police.Name = "Police, The";

    var avril = db.Artists.Single(a => a.Name == "Avril Lavigne");
    db.Artists.Remove(avril);

    db.SaveChanges();
}
{% endhighlight %}

Conclusion
==========
Hopefully by now, you have enough information to get started using the Entity Framework with a MySQL database. For many,
many more articles on how to use EF, check out our team's official [Getting Started][9] page on MSDN.


  [1]: http://msdn.com/data/ef
  [2]: http://www.mysql.com
  [3]: http://msdn.microsoft.com/en-us/data/jj193542
  [4]: http://dev.mysql.com/downloads/connector/net
  [5]: http://www.devart.com/dotconnect/mysql
  [6]: http://en.wikipedia.org/wiki/Free_and_open-source_software
  [7]: http://chinookdatabase.codeplex.com
  [8]: {% post_url 2012-05-10-using-entity-framework-code-first-with %}
  [9]: http://msdn.microsoft.com/en-us/data/ee712907
