---
layout: post
title: Image Resizer Powertoy Clone 2.0 in the Works
permalink: /2009/01/image-resizer-powertoy-clone-20-in.html
tags: image-resizer
---

Over this past week, I have been working hard to get my [Image Resizer Powertoy Clone for Windows][1] working on
Windows Vista 64-bit. I have tried every tweak I could think of to get the current version working, but to no avail.

I believe the problem is due to the fact that the project is written in C#/.NET. When a .NET Assembly is registered as a
COM object, it uses mscoree.dll as the in-process server to handle communication between COM clients and the .NET
Assembly. I suspect that <del>this file is a 32-bit DLL and, thus, cannot be loaded into the 64-bit explorer.exe</del>.
(**Update**: That statement wasn't true.) ...or at least that's my theory.

I have taken this opportunity to dust off my C++/Win32 skills. I am rewriting the entire application in C++ (the way it
should have been done in the first place). In a few weeks, I expect to have a working 64-bit version of the shell
extension. I am also going to further investigate problems people have had with it not working in versions of Windows
prior to XP (I know what the problem is, but I never could get it resolved).

Along with these new upgrades, I am playing with the idea of moving the project to [CodePlex][3]. It's kind of ironic
that I would let Microsoft host a project I stole from them, but [SourceForge][4] is a nightmare to deal with these
days.


  [1]: http://sourceforge.net/projects/phototoysclone
  [2]: http://www.microsoft.com/windows/windows-vista/compare-editions/64-bit.aspx
  [3]: http://www.codeplex.com
  [4]: http://sourceforge.net
