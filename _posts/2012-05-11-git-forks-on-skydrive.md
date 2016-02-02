---
layout: post
title: Git forks on SkyDrive
permalink: /2012/05/git-forks-on-skydrive.html
tags: git
---

The other day, I found myself carrying a USB flash drive between two of my computers so that I could continue working on
some code that I had been writing. Every time you find yourself touching a USB flash drive, I want you to stop
immediately, look into an imaginary camera, and exclaim, "To the cloud!"

Most project hosters like CodePlex, GitHub, etc. let you create a fork on their servers to share changes between
computers. The code I was working on, however, wasn't being hosted.

To the cloud!
=============
The [SkyDrive apps][1] for Windows and Mac create a special directory on your computer that automatically gets synced to
SkyDrive and your other computers. I decided to create my fork there. Here is an example of forking my [Image
Resizer][2] project to SkyDrive.

```
# Create a SkyDrive fork
cd %USERPROFILE%\SkyDrive
git clone --bare https://git01.codeplex.com/imageresizer
```

Note that this is a bare repository. Don't edit these files directly.

After creating the fork, add it as a remote to your existing clone. Here is an example of creating another clone of the
ImageResizer repository and adding the skydrive remote. Do this outside the SkyDrive directory.

```
# Add SkyDrive fork as a remote to a new clone
git clone https://git01.codeplex.com/imageresizer
cd imageresizer
git remote add skydrive %USERPROFILE%\SkyDrive\imageresizer.git
```

To share work between computers, push your changes to the fork.

```
# Push to SkyDrive fork
git push skydrive work
```

To get those changes on another computer, wait for SkyDrive to finish syncing and pull them.

```
# Pull from SkyDrive fork
git pull skydrive work
```

There you have it, your very own SkyDrive fork! Now, what else can we do to eliminate that USB flash drive from our
lives for good?


  [1]: https://apps.live.com/skydrive
  [2]: http://imageresizer.codeplex.com
