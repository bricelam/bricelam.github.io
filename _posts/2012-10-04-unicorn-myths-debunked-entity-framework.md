---
layout: post
title: "Unicorn Myths Debunked: Entity Framework 4.4"
permalink: /2012/10/unicorn-myths-debunked-entity-framework.html
tags: entity-framework
---

Caution, this post is a bit of a rant. I continue to see people in the community referring to something called
*Entity Framework 4.4*, and I just want to set the record straight.

<span style="color: red; font-size: x-large;">There is no such thing as Entity Framework 4.4!</span>

The 4.4 comes from the assembly version of EntityFramework.dll when you install EntityFramework 5.0 into a project that
targets .NET Framework 4.0. This is merely a side effect of how the runtime loads and binds to assemblies, and in no way
reflects the version of the product.

An experiment
=============
Here is some simple code that prints out version information about the Entity Framework assembly.

{% highlight csharp %}
var assembly = typeof(DbContext).Assembly;
var name = assembly.GetName();
var info = FileVersionInfo.GetVersionInfo(assembly.Location);

Console.WriteLine("Assembly Version: {0}", name.Version);
Console.WriteLine("Product Version: {0}", info.ProductVersion);
{% endhighlight %}

When we run the code on an application targeting .NET Framework 4.5, we get the following output.

    Assembly Version: 5.0.0.0
    Product Version: 5.0.0.net45

On .NET Framework 4.0, we get:

    Assembly Version: 4.4.0.0
    Product Version: 5.0.0.net40

As you can see, the product version in both cases is 5.0.

In conclusion
=============
Please always refer to the product as Entity Framework 5.0 and, if you think the information is relevant, also let
people know what version of the .NET Framework you are targeting.
