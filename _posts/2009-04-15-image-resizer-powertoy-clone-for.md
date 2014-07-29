---
layout: post
title: "Image Resizer Powertoy Clone for Windows: The First Year"
permalink: /2009/04/image-resizer-powertoy-clone-for.html
tags: image-resizer
---

Last Saturday was the one-year anniversary of my [Image Resizer Powertoy Clone for Windows][1] project. I thought it
would be fun to talk about some of the project's highlights of the past year.

A few years ago, my mother asked me if there was an easier way to resize photos. My dad had walked her through the steps
of doing it one picture at a time through Photoshop, but that was such a tedious process. I, being the wonderful son
that I am, introduced her to Microsoft's [Image Resizer Powertoy for Windows XP][2]. A few years later however, she
decided to get a new laptop with Windows Vista. She, like myself and so many others, was disappointed to find that the
Powertoy would no longer work. Thus, I set off to change the world (or at least the world of image resizing).

The first step, as any great developer knows, was to see if it had already been done. There were a few other tools out
there, but none of them really stood up to the original. I decided to develop it in C#/.NET (mostly I was just curious
to see if it was even possible). I immediately hit a dead end. After a few weeks of searching however, I found [an
article][3] that talked about Windows Shell Extension in C#. The article was bit dated, but using the [MSDN articles][4]
I was able to get it working.

I decided to post the project on SourceForge.net because of the hard time I had finding examples of .NET Shell
Extensions. Well, that and I knew that if I had any chance of it becoming popular, I would need a big name like
SourceForge to help. I also knew that to succeed, people had to be able to find the thing. The name, I decided, needed
to be as close to the original's as possible. So I took the original, *Image Resizer Powertoy for Windows XP*, dropped
the *XP* (as it no longer applied), and added the word *Clone* (to avoid getting sued by Microsoft :wink:). The final
step of the release was getting it to show up on search engines. I posted a Digg article titled [Image Resizer PowerToy
for Windows Vista][5] to hopefully spark some online gossip. Sure enough, the download counts started growing, and I
slowly moved up the page on search results.

A couple months after the initial release, I got my first user-submitted issue. [Earthclimate][6] asked me to [please
maintain metadata][7]. He discovered that, after resizing an image, all the metadata (e.g. date taken, tags, rating,
camera type, f-stop, ISO settings, etc.) disappeared. I immediately fixed this and released the bits. Isn't open source
wonderful?

The next major spike in downloads, I attribute to a blog post and YouTube upload about my project by [PCWizKid][8]. The
month [Windows Vista - Free Image Resizer][9] was posted, I saw a 200% jump in downloads.

With more popularity comes more scrutiny. [Rshelq][10] submitted an issue regarding the application's [poor image
quality][11]. Together, we optimized the resizing process to produce an image that was, according to rshelq, better than
the software which I set out to clone. With the new changes came another 25% jump in downloads. The biggest jump in
downloads, however, came in December. From November to December, there was a 100% jump (I know, 100 is smaller than 200
but these are increases in the number of downloads per month, not just increases in the number of downloads). Since
December, the project has averaged about one download every six minutes!

One of the most frequent complaints I received was that my application didn't work on 64-bit editions of Windows. After
seven months, I decided it was time to tackle the beast. After days and days of digging and hundreds of dollars later
(because of course, I purchased my copy of Windows Vista 64-bit), I decided that Windows Shell Extensions, written in
.NET, don't work on 64-bit editions of Windows simply because: **the coding gods forbid it**. Yes, this is the most
logical explanation I have. Despite everything being in place so that it *should* work, it just doesn't. So I did what I
always do when I get stuck on a project: I started over from scratch. For the second version, I decided to listen when
they said to never write Shell Extensions in .NET, and I did it the right way - in good ol' C++. This was a fun
experience; it had been nearly three years since I last touched C++, and I had to relearn pretty much everything about
the language. I had also never done WIN32 or COM development (cryptic technologies that have been around for ages).
However, using my amazing ability to learn new things (or maybe just my amazing ability to sift through pages and pages
of dry technical documentation), I was able to get it working. There are still a few things left to do before I consider
version 2.0 done (like proper error handling, and scrapping the dependency on MFC which adds nearly 12MB to the
download), but the Beta seems to be flourishing well with over 16,000 downloads (2,500 of which are for the 64-bit
edition).

So, in celebration of a great first year, I want to thank everyone who has helped contribute to the success of this
little project. So far, the project has been downloaded over 50,000 times! If you're interested in contributing, you can
do so by spreading the word (like all those awesome bloggers out there who [make my project look sexy][12], submitting a
[feature request or bug report][13], or simply by [downloading][14] and using it. The project, which was migrated to
[CodePlex][15] due to the degrading quality of SourceForge's services, can be found at
<http://phototoysclone.codeplex.com>. Enjoy, and happy resizing!


  [1]: http://phototoysclone.codeplex.com
  [2]: http://www.microsoft.com/windowsxp/using/digitalphotography/learnmore/tips/eschelman2.mspx
  [3]: http://www.theserverside.net/tt/articles/showarticle.tss?id=ShellExtensions
  [4]: http://msdn.microsoft.com/en-us/library/bb776881.aspx
  [5]: http://digg.com/microsoft/Image_Resizer_PowerToy_for_Windows_Vista
  [6]: http://sourceforge.net/users/earthclimate
  [7]: http://sourceforge.net/tracker/?func=detail&aid=1994203&group_id=224498&atid=1061604
  [8]: http://pcwizkid.blogspot.com
  [9]: http://www.youtube.com/watch?v=pUxHQAnWRC0
  [10]: http://sourceforge.net/users/rshelq
  [11]: http://sourceforge.net/tracker/?func=detail&aid=2227464&group_id=224498&atid=1061604
  [12]: http://nerwin.net/2009/04/image-resizer-powertoy-for-vista
  [13]: http://phototoysclone.codeplex.com/WorkItem/List.aspx
  [14]: http://phototoysclone.codeplex.com/Release/ProjectReleases.aspx
  [15]: http://www.codeplex.com
