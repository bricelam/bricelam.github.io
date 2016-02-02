---
layout: post
title: System.Data.SQLite on Entity Framework 6
permalink: /2013/06/systemdatasqlite-on-entity-framework-6.html
tags: entity-framework sqlite
---

<div style="color: red; text-align: center;">
  <hr />
  <span style="font-size: large;">This feature has been added.</span><br />
  Versions 1.0.91.0 and newer of System.Data.SQLite support EF6, this fork is not required.
  <hr />
</div>

If you've tried my tutorial for using [Entity Framework on SQLite][1], you may have noticed that it doesn't work on
[Entity Framework 6][2]. If you set everything up just like you're supposed to, you still get the following error.

> The 'Instance' member of the Entity Framework provider type 'System.Data.SQLite.SQLiteProviderServices,
> System.Data.SQLite.Linq, Version=1.0.86.0, Culture=neutral, PublicKeyToken=db937bc2d44ff139' did not return an object
> that inherits from 'System.Data.Entity.Core.Common.DbProviderServices'. Entity Framework providers must extend from
> this class and the 'Instance' member must return the Singleton instance of the provider.

This is because there have been some [breaking changes to the provider model in Entity Framework 6][3], and providers
will need to be updated before they can work. (See the linked article for more information.)

An updated provider
===================
Since the [System.Data.SQLite][4] provider is open source, I took the liberty of creating a fork and updating it to work
with Entity Framework 6. You can find it hosted over on GitHub - the [bricelam/System.Data.SQLite.Linq][5] repository.

This is not in any way an official or supported fork; I just wanted to get something out there for the community to play
with. Also, I do not intend to submit a pull request for my changes since they very trivial, and the more significant
work will be integrating it into their build system.

Installing it
=============
How can you get it? Easy, I've uploaded it to a NuGet feed hosted over on [MyGet][6]. You can install it directly from
[Package Manager Console][7] using the following.

```powershell
Install-Package System.Data.SQLite.Linq -Pre -Source
    https://www.myget.org/F/bricelam/
```

Enjoy! And be sure to give us any feedback regarding EF6 over on [our CodePlex project page][8].


  [1]: {% post_url 2012-10-12-entity-framework-on-sqlite %}
  [2]: http://blogs.msdn.com/b/adonet/archive/2013/05/30/ef6-beta-1-available.aspx
  [3]: https://entityframework.codeplex.com/wikipage?title=Rebuilding%20EF%20providers%20for%20EF6
  [4]: http://system.data.sqlite.org
  [5]: https://github.com/bricelam/System.Data.SQLite.Linq
  [6]: http://myget.org
  [7]: http://docs.nuget.org/docs/start-here/using-the-package-manager-console
  [8]: https://entityframework.codeplex.com
