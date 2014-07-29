---
layout: post
title: Image Resizer 2.1
permalink: /2009/12/image-resizer-21.html
tags: image-resizer
---

[Version 2.1][1] of the [Image Resizer Powertoy Clone for Windows][2] was actually released on Friday, December 11th,
but I'm just now getting around to writing about it. (I've been sick for a few days.)

This was a particularly tricky release to plan. The decisions of what to include in this release were governed by the
following principles:

* Release as soon as possible
* Only include work items that affect a majority of the users
* Don't introduce any new UI elements
* Maintain compatibility with Microsoft-supported versions of Windows

The first of these (release as soon as possible) is pretty straightforward; I wanted to get new bits out and quick. This
manifested by cutting out time-consuming work items like [Multi-Language Support][3]. I wasn't about to wait for the
utility to be translated into 11+ different languages before releasing some of the more critical changes.

Only including work items that affected a majority of the users basically means that I want to tackle some of the most
requested enhancements and most encountered problems. The most requested features were [Remember Resize Settings][4] and
[Update/Modernize Default Sizes][5]. Of course, some users requested these in very strange ways such as doing away with
the Basic view, or requesting that I add their specific size to the default list of sizes.

The [Auto-Width/Height][6] option wasn't really a request, but more of a compromise. Many users requested that I change
the resizing behavior to match that of Microsoft's version of the tool. Personally, I believe that the way Microsoft did
it is wrong: The UI read that the new image would *fit* a \__\__x____ screen, but in reality it would resize the new
image to *fill* a screen of the specified dimensions. I think that most users expect a resized image to fit within the
dimensions they specify, so I refused to bring back the original's behavior. After many heated discussions, I realized
that it wasn't the old behavior that they wanted at all, but instead they just wanted to be able to resize an image to
fill a specified width or height. Essentially, the old tool would just take the biggest of the numbers you entered and
use it (which makes me wonder why it forced you to enter two in the first place). So I decided to give users the ability
to just enter one of the width or height, and the other would be calculated accordingly.

The biggest thing, by far, that I wanted to fix with this release was in fact a user error. I received a ton of feedback
from users that my tool was a huge disappointment because it didn't work. As I started looking into all these cases, I
began to realize where my tool was breaking: it wasn't. Users were ignorantly installing the 32-bit version of the
utility on their 64-bit version of Windows. This is a perfectly legitimate scenario, but the effect is that the
*Resize Pictures* option will only appear in 32-bit programs. They would browse to their pictures' folder in Windows
Explorer (which would be a 64-bit program) and right-click, but the option to resize wouldn't be there. They would then
proceed to get angry and find a way to vent by leaving hate comments on blog posts or in [my anonymous survey][7] or by
reviewing my software poorly because it doesn't work for them. Naturally, this pissed me off so, due to these stupid
users - only those who got angry are stupid - I decided to disallow 32-bit installs on a 64-bit operating system.

There were many other requests that I received; however, I also got a lot of feedback from users saying that, "It's
perfect the way it is" and, "Avoid making it bloated," so I tried to keep the new features to a minimum for this
release.

Anyway, back to my governing principles. My reluctance to introduce any new UI elements was two-fold. First, as I just
mentioned, there were a lot of users that praised the simplicity of the tool and cautioned me against adding a bunch of
bells and whistles. The second reason that I was against it is because I have a lot of international users. Seeing how
the tool is currently only available in English, I'm sure that many people have learned to use it by memorization.
Introducing anything new or changed options would probably just confuse these users.

Finally, I wanted to make sure my tool ran on Microsoft-supported versions of Windows. This, of course, meant including
a fix for the [Pre-XP Support][8] issue. This principle though actually ended up being more restrictive than I would
have liked. The tool is for a UI overhaul to bring it up to the [Windows Vista/7 standard][9], but many of these things
are not available in previous versions of Windows. I would also like to change the resizing process to make use of
[Direct2D][10] and the [Windows Imaging Component][11], but again they're not backwards compatible. This will probably
continue to be a guiding principle for the next couple of releases, but after that I fully intend to break backwards
compatibility in favor of new, innovative technologies.

All in all, I'm very proud of this release. The code has become drastically cleaner as I have learned considerably more
about C++, Win32, and ATL development. The seemingly little new features pack quite a punch and make the tool a lot
friendlier to use. Go [download it now][1], and tell all your friends. Expect to see [Multi-Language Support][3] in the
next release, and after that, prepare to have your minds blown...


  [1]: http://imageresizer.codeplex.com/Release/ProjectReleases.aspx?ReleaseId=30247
  [2]: http://imageresizer.codeplex.com
  [3]: http://imageresizer.codeplex.com/WorkItem/View.aspx?WorkItemId=2225
  [4]: http://imageresizer.codeplex.com/WorkItem/View.aspx?WorkItemId=2151
  [5]: http://imageresizer.codeplex.com/WorkItem/View.aspx?WorkItemId=3167
  [6]: http://imageresizer.codeplex.com/WorkItem/View.aspx?WorkItemId=3206
  [7]: http://spreadsheets.google.com/viewform?formkey=dGluVGxIOUY5X1ZsMEhoZmI3anp0RXc6MA..
  [8]: http://imageresizer.codeplex.com/WorkItem/View.aspx?WorkItemId=2437
  [9]: http://msdn.microsoft.com/en-us/library/aa511258.aspx
  [10]: http://msdn.microsoft.com/en-us/library/dd370987.aspx
  [11]: http://msdn.microsoft.com/en-us/library/ee719902.aspx
