---
layout: post
title: Paket-like NuGet with MSBuild
tags: msbuild
---

Say what you'd like about XML vs. JSON. (I like [YAML][1]) I'm still very excited about [NuGet's deeper integration with
MSBuild][2]. That's because MSBuild is so much more than its antiquated file format. It's an execution engine that
enables you to define configuration and sets of data, transform that data, then process it using various tasks.

I'm also a big fan of [Paket][3], so I was intrigued when @llehn asked:

> Is there something to prevent me from adding two different versions of same library? Like Paket which does global
> dependency resolving.

In this post, I'd like to walk you through a prototype of how to structure your NuGet references to resemble
[paket.dependencies][4] and [paket.references][5] files.

The file layout looks like this:

    .
    ├───App.sln
    ├───Dependencies.props
    ├───...
    ├───App
    │   ├───App.csproj
    │   ├───References.props
    │   └───...
    └───...

`Dependencies.props` declares any dependencies that you'll need throughout your solution. It goes in the root directory
next to your solution file and looks like this:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <ItemGroup>
    <Depencency Include="Newtonsoft.Json" Version="9.0.*"/>
    <Depencency Include="xunit" Version="2.2.0-*"/>
  </ItemGroup>

  <!-- Transform any referenced dependencies into actual NuGet package references just
       before they're needed -->
  <Target Name="ResolveDepencencyRefs" BeforeTargets="_GetProjectRestoreType">
    <ItemGroup>
      <PackageReference
          Include="@(Depencency)"
          Condition="'@(Depencency)' == '@(DepencencyRef)' and '%(Identity)' != ''"/>
    </ItemGroup>
  </Target>
</Project>
```

The `*.csproj` files would simply reference their corresponding `References.props` file.

```xml
<Import Project="References.props"/>
```

`References.props` references the dependencies that are used by that project:

```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Import Project="$(SolutionDir)Dependencies.props"/>

  <ItemGroup>
    <DepencencyRef Include="Newtonsoft.Json"/>
    <DepencencyRef Include="xunit"/>
  </ItemGroup>
</Project>
``` 

And that's it! Your dependency versions are defined in a single place for your entire solution, and each project
defines which dependencies it needs.

This is just one example of a new scenario that can be enabled by moving NuGet package references into MSBuild. I look
forward to seeing what else the community comes up with.


  [1]: http://yaml.org/
  [2]: https://blogs.msdn.microsoft.com/dotnet/2016/10/19/net-core-tooling-in-visual-studio-15/
  [3]: http://fsprojects.github.io/Paket/
  [4]: http://fsprojects.github.io/Paket/dependencies-file.html
  [5]: http://fsprojects.github.io/Paket/references-files.html
