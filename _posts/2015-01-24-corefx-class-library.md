---
layout: post
title: '.NET Core Class Libraries'
---

I'm super excited about .NET Core. If you haven't yet, I suggest you read [Introducing .NET Core][1] by Immo Landwerth
(@terrajobst). It paints a beautiful vision of the future in which different .NET platforms (e.g. .NET Framework,
Windows Phone, Windows Store, etc.) are based on a single, contract-based, NuGet-delivered framework: .NET Core.

How do I use it?
================
Unfortunately, there aren't any tools to help you create .NET Core class libraries yet. The ASP.NET 5 tools inside
Visual Studio 2015 allow you to create ASP.NET Core class libraries, but these are subtly different. They are .NET Core
class libraries that are only intended to be run on ASP.NET 5. <span style="text-decoration: line-through;">A special compile step is performed that can make your
assemblies incompatible with other .NET Core platforms</span>. (**Update:** This feature has been removed) The tools also don't enable you to create portable class
libraries, but instead cross-compile to each platform (similar to how shared projects work). <span style="text-decoration: line-through;">They also don't enable you
to strong-name the assemblies.</span> (**Update:** This feature is being added)

One assembly to rule them all
=============================
So how can you create a single, portable assembly that can run on all .NET Core platforms? Let me show you.

Start by creating a portable class library that that targets .NET Framework 4.5 and Windows 8.

![Add Portable Class Library]({{ "/attachments/AddPCL.png" | prepend: site.baseurl | prepend: site.url }})

The targets don't actually matter so long as you can install NuGet packages that target
`portable-wpa81+wp80+win80+net45+aspnetcore50` like the .NET Core packages currently do.

Hack it good
------------
Now for the magic part. Edit the .csproj file and add the following fragment inside of the first `PropertyGroup`
element.

{% highlight XML %}
<ImplicitlyExpandTargetFramework>false</ImplicitlyExpandTargetFramework>
{% endhighlight %}

If you try to build now, you should get several errors including my favorite:

> Predefined type 'System.String' is not defined or imported

How is this possible? Well, you've created a project that doesn't reference any assemblies. That includes
`System.Runtime` and `mscorlib`. This same trick is use by the [dotnet/corefx][2] repository to build the .NET Core
assemblies.

Add references
==============
How do you reference .NET Core? Easy, install NuGet packages! To get the empty class library project to compile again,
type the following inside of [Package Manager Console][3].

{% highlight PowerShell %}
Install-Package System.Resources.ResourceManager -Prerelease -DependencyVersion Highest
Install-Package System.Linq -Prerelease -DependencyVersion Highest
{% endhighlight %}

(Note, the `DependencyVersion` arguments ensure you get a consistent build of .NET Core.)

All done
========
That's it! Your single assembly should now be able to run on all the .NET Core platforms including .NET Framework 4.6,
.NET Native, ASP.NET 5 & ASP.NET Core 5.

You can even reference *new* contracts like `System.IO.FileSystem` and `System.Threading.Thread`.

Happy coding.


  [1]: http://blogs.msdn.com/b/dotnet/archive/2014/12/04/introducing-net-core.aspx
  [2]: https://github.com/dotnet/corefx
  [3]: http://docs.nuget.org/docs/start-here/using-the-package-manager-console
