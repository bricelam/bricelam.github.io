---
layout: post
title: Server Project Ideas
permalink: /2008/10/server-project-ideas.html
---

About a month ago, I switched my server from [Gentoo Linux][1] to [Windows Home Server][2]. I have quite enjoyed the new
platform. I love how tightly it integrates with my other Windows desktops (and my Xbox ...if I had one). I can pull up
Windows Media Player and access any media on the server directly. There is also an external web page that allows me to
connect to the server or other desktops on the network remotely. I have even found a plugin ([Whiist][3]) that enables
easy configuration of the web server. All in all, I am very happy with the switch.

There are only a few things that are missing now. I would like a way to schedule tasks from the remote console, a way to
download torrents remotely, and a way to add code repositories. So, I am proposing three projects.

The scheduled tasks add-in should be fairly straight forward. Any administrator can log into the remote console, see a
list of scheduled tasks, configure them, or add new ones. I don't think that non-administrative users need to be able to
do this as it is an advanced feature. My main uses for this would be capture snapshots of external databases or code
repositories for back-up purposes, but it could also be useful to push things to other servers or just perform periodic
maintenance.

The torrent add-in would be the biggest project of the three, you would need to be able to configure the torrent client
from the administrative console, and perhaps add permissions for which users can use it. An authenticated web
application is also needed. This would allow users to manage thier torrents, add new ones, or just download the
completed torrent. I believe that a windows service may also be necessary so the two can properly communicate, but I
guess you could just have the ASP.NET application start and manage the torrent client and have a way to restart it from
the console when settings are changed.

Finally, the code repositories. It would be easy enough to set up a [Subversion][4] server on the machine, but I would
also want an easy way to add new repositories, and manage user permissions.

I was also going to add a database management plugin to the list, but after I thought about it, most DBMS are already
meant to be managed remotely, so it would just be a matter of installing it.

Anywho, these are just my thoughts. I'm sure I'll play around a bit in my spare time and see how far I can get, if I'm
proud of my work, you can expect them to be posted somewhere (probably [CodePlex][5] because it rocks).


  [1]: http://www.gentoo.org
  [2]: http://windows.microsoft.com/en-us/windows/windows-home-server
  [3]: http://www.andrewgrant.org/whiist
  [4]: http://subversion.tigris.org
  [5]: http://www.codeplex.com
