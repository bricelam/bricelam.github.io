---
layout: post
title: T4 and EF Core Reverse Engineering
tags: [ entity-framework, t4 ]
---

I'm a huge fan of T4--the underutilized [templating engine](https://docs.microsoft.com/en-us/visualstudio/modeling/code-generation-and-t4-text-templates) that ships as part of Visual Studio.

Entity Framework 6 allows you to customize the code generated for a model using T4 templates, and I've been trying to bring bring this functionality into EF Core since the beginning.

I finally got around to putting together a sample--in the [bricelam/EFCore.TextTemplating](https://github.com/bricelam/EFCore.TextTemplating) repo--that demonstrates how to use T4 to customize the code scaffolded by EF Core. Technically, this has been possible since EF Core 2.1.

Here's an example of a T4 template that generates a DbContext class:

<div class="language-tt highlighter-rouge"><div class="highlight"><pre class="highlight"><code><span class="nt">&lt;#@ template</span> <span class="na">language=</span><span class="s">"C#"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"Model"</span> <span class="na">type=</span><span class="s">"Microsoft.EntityFrameworkCore.Metadata.IModel"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"ModelNamespace"</span> <span class="na">type=</span><span class="s">"System.String"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"Namespace"</span> <span class="na">type=</span><span class="s">"System.String"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"ContextName"</span> <span class="na">type=</span><span class="s">"System.String"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"ConnectionString"</span> <span class="na">type=</span><span class="s">"System.String"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"Code"</span> <span class="na">type=</span><span class="s">"Microsoft.EntityFrameworkCore.Design.ICSharpHelper"</span> <span class="nt">#&gt;</span>
<span class="nt">&lt;#@ parameter</span> <span class="na">name=</span><span class="s">"ProviderCode"</span> <span class="na">type=</span><span class="s">"Microsoft.EntityFrameworkCore.Scaffolding.IProviderConfigurationCodeGenerator"</span> <span class="nt">#&gt;</span>
<span class="go">using Microsoft.EntityFrameworkCore;
using</span> <span class="nt">&lt;#=</span> <span class="n">ModelNamespace</span> <span class="nt">#&gt;</span><span class="go">;

namespace</span> <span class="nt">&lt;#=</span> <span class="n">Namespace</span> <span class="nt">#&gt;</span>
<span class="go">{
    public partial class</span> <span class="nt">&lt;#=</span> <span class="n">ContextName</span> <span class="nt">#&gt;</span> <span class="go">: DbContext
    {</span>
<span class="nt">&lt;#</span>
    <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">entityType</span> <span class="k">in</span> <span class="n">Model</span><span class="p">.</span><span class="nf">GetEntityTypes</span><span class="p">())
    {</span>
<span class="nt">#&gt;</span>
        <span class="go">public DbSet&lt;</span><span class="nt">&lt;#=</span> <span class="n">entityType</span><span class="p">.</span><span class="n">Name</span> <span class="nt">#&gt;</span><span class="go">&gt;</span> <span class="nt">&lt;#=</span> <span class="n">entityType</span><span class="p">[</span><span class="s">"Scaffolding:DbSetName"</span><span class="p">]</span> <span class="nt">#&gt;</span> <span class="go">{ get; set; }</span>
<span class="nt">&lt;#</span>
    <span class="p">}</span>
<span class="nt">#&gt;</span>

        <span class="go">protected override void OnConfiguring(DbContextOptionsBuilder options)
            =&gt; options</span><span class="nt">&lt;#=</span> <span class="n">Code</span><span class="p">.</span><span class="nf">Fragment</span><span class="p">(</span><span class="n">ProviderCode</span><span class="p">.</span><span class="nf">GenerateUseProvider</span><span class="p">(</span><span class="n">ConnectionString</span><span class="p">)</span><span class="p">)</span> <span class="nt">#&gt;</span><span class="go">;

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {</span>
<span class="nt">&lt;#</span>
    <span class="k">foreach</span> <span class="p">(</span><span class="kt">var</span> <span class="n">entityType</span> <span class="k">in</span> <span class="n">Model</span><span class="p">.</span><span class="nf">GetEntityTypes</span><span class="p">())
    {</span>
<span class="nt">#&gt;</span>
            <span class="go">modelBuilder.ApplyConfiguration(new</span> <span class="nt">&lt;#=</span> <span class="n">entityType</span><span class="p">.</span><span class="n">Name</span> <span class="nt">#&gt;</span><span class="go">Configuration());</span>
<span class="nt">&lt;#</span>
    <span class="p">}</span>
<span class="nt">#&gt;</span>
        <span class="go">}
    }
}</span>
</code></pre></div></div>

⚠**Warning!** Staring directly at T4 templates without syntax highlighting may hurt your eyes. I strongly recommend using the [Devart T4 Editor](https://marketplace.visualstudio.com/items?itemName=DevartSoftware.DevartT4EditorforVisualStudio) for Visual Studio or [T4 Support by Zachary Becknell](https://marketplace.visualstudio.com/items?itemName=zbecknell.t4-support) for VS Code.

I took this opportunity to fix a few of the pet peeves I have with EF Core's built-in scaffolding: First, it only generates validation attributes since these are actually used by other technologies like ASP.NET Core. Generating data annotations only used by EF always seemed a bit silly to me. Second, it only generates the parts of the model that affect EF's behavior. Things like sequences, foreign key constraint names, and non-unique indexes just clutter the model in my opinion. (Note, these things are useful in Migrations, just not in a database-first workflow.) Finally, it generates configuration classes to keep OnModelCreating from becoming exceedingly large and unmaintainable. But please remember, these templates are merely a starting point. Feel free to tweak them to your heart's content.

To get started, clone the repository and copy the EFCore.TextTemplating project into your solution. You'll also need to add an assembly-level attribute to your app:

``` cs
[assembly: DesignTimeServicesReference(
    "EFCore.TextTemplating.DesignTimeServices, EFCore.TextTemplating")]
```

After doing this, the templates should now be used whenever you reverse engineer a model from the database:

``` sh
dotnet ef dbcontext scaffold "Data Source=Chinook.db" Microsoft.EntityFrameworkCore.Sqlite
```

If you're not using Visual Studio, you'll need to use [dotnet-ef](https://github.com/mono/t4) after editing the template files to re-generate the code-behind files:

``` sh
dotnet tool install -g dotnet-t4

t4 MyDbContextGenerator.tt -c MyDbContextGenerator -o MyDbContextGenerator.cs
t4 MyEntityTypeConfigurationGenerator.tt -c MyEntityTypeConfigurationGenerator -o MyEntityTypeConfigurationGenerator.cs
t4 MyEntityTypeGenerator.tt -c MyEntityTypeGenerator -o MyEntityTypeGenerator.cs
```

And that's it, you now have full control over the scaffolded code. You can do things like define how table and column names map to class and property names, add a dictionary to define navigation property names based on a foreign key name, or finally fix all those analyzer issues we that we keep closing--it's just C#!
