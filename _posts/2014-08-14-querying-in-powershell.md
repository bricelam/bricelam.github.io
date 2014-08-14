---
layout: post
title: Querying in PowerShell
tags: powershell
---

Thanks to my work on [Entity Framework][1]'s [Code First Migrations][2] [NuGet][3] PowerShell commands, I've gained
quite a bit of experience programming in PowerShell. In this post, I want to show you some of PowerShel's query
operators.

In PowerShell, you pipe commands together and pump data through them to get a result. In order to do this, we need some
data. Here's some data loosely based on the [Chinook Database][4], my favorite sample database.

{% highlight powershell %}
$albums = @(
    [PSCustomObject]@{
        Artist = 'Anberlin'
        Title = 'Never Take Friendship Personal'
        Tracks = 11
    }
    [PSCustomObject]@{
        Artist = 'Anberlin'
        Title = 'Cities'
        Tracks = 12
    }
    [PSCustomObject]@{
        Artist = 'Angels & Airwaves'
        Title = 'We Don''t Need to Whisper'
        Tracks = 10
    }
    [PSCustomObject]@{
        Artist = 'Something Corporate'
        Title = 'Leaving Through The Window'
        Tracks = 14
    }
)
{% endhighlight %}

Projecting
==========
To project only certain properties of objects, use the `select` command. Here is an example of selecting just the Artist
properties.

{% highlight powershell %}
$albums | select Artist
{% endhighlight %}

You can select only distinct results using the `-Unique` parameter.

{% highlight powershell %}
$albums | select Artist -Unique
{% endhighlight %}

Paging
======
You can also page results using the `select` command's `-Skip`, `-First`, and `-Last` parameters. Here's how to get the
first album.

{% highlight powershell %}
$albums | select -First 1
{% endhighlight %}

Here's how to get the second album.

{% highlight powershell %}
$albums | select -First 1 -Skip 1
{% endhighlight %}

Here's how to get the last one.

{% highlight powershell %}
$albums | select -Last 1
{% endhighlight %}

Filtering
=========
Filtering, like in most languages, is done using the `where` command. The following is an example of selecting all the
albums with an artist who's name starts with an *A*.

{% highlight powershell %}
$albums | where Artist -like 'A*'
{% endhighlight %}

You can also do it as an expression which allows for more complex filtering.

{% highlight powershell %}
$albums | where { $_.Artist.StartsWith('A') }
{% endhighlight %}

Ordering
========
To order the results, use the `sort` command. Let's order the albums by number of tracks.

{% highlight powershell %}
$albums | sort Tracks
{% endhighlight %}

Now, let's do it in descending order.

{% highlight powershell %}
$albums | sort Tracks -Descending
{% endhighlight %}

Grouping
========
Grouping is accomplished using the `group` command. Let's group the albums by artist.

{% highlight powershell %}
$albums | group Artist
{% endhighlight %}

Aggregating
===========
Basic aggregation can be done using the `measure` command.

Use the `-Average` parameter to get the average.

{% highlight powershell %}
$albums | measure Tracks -Average
{% endhighlight %}

To get the maximum value use `-Maximum`.

{% highlight powershell %}
$albums | measure Tracks -Maximum
{% endhighlight %}

To get the maximum value use `-Maximum`.

{% highlight powershell %}
$albums | measure Tracks -Minimum
{% endhighlight %}

Finally, to get the summation of all values, use `-Sum`.

{% highlight powershell %}
$albums | measure Tracks -Sum
{% endhighlight %}

More
====
This post is merely an introduction to the ways you can query over data in PowerShell. For more inforamtion, see the
official documentation for the following commands.

* [ForEach-Object][5]
* [Group-Object][6]
* [Select-Object][7]
* [Sort-Object][8]
* [Where-Object][9]


  [1]: http://msdn.com/data/ef
  [2]: http://msdn.microsoft.com/en-us/data/jj591621
  [3]: http://nuget.org
  [4]: http://chinookdatabase.codeplex.com
  [5]: http://technet.microsoft.com/en-US/library/hh849731.aspx
  [6]: http://technet.microsoft.com/en-US/library/hh849907.aspx
  [7]: http://technet.microsoft.com/en-us/library/hh849895.aspx
  [8]: http://technet.microsoft.com/en-US/library/hh849912.aspx
  [9]: http://technet.microsoft.com/en-US/library/hh849715.aspx
