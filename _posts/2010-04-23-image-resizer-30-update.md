---
layout: post
title: Image Resizer 3.0 Update
permalink: /2010/04/image-resizer-30-update.html
tags: image-resizer
---

A member of the CodePlex team at Microsoft recently contacted me. He asked me if I would be interested in allowing
CodePlex and PreEmptive Solutions to use [my project][1] as a showcase for an upcoming integration between their two
companies. I was flattered by this opportunity; however, I realized that in order to participate, I would have to
rewrite a majority of my project into .NET. Being the ambitious person I am (or perhaps just because I'm a Microsoft
whore), I decided to go for it.

Originally, I had planned on working my way up to version 3.0 of Image Resizer. Version 2.2 was going to be a
multi-lingual bug fix release, and version 2.3 would finally introduce some of my most requested new features. But, in
light of recent events, I am going to forgo these releases and dive straight into my 3.0 vision.

I've been racking my brain on how best to introduce the new features, and I think things are starting to come together
nicely. There will be two main applications: the Image Resizer, and it's Settings. The Image Resizer is basically what
the application is today with a major UI overhaul and some subtle new features. It will be a multi-page dialog
consisting of the following page, a progress page, and a results page.

![Image Resizer 3 Prototype]({{ "/attachments/ImageResizer.png" | absolute_url }})

Please note that this screenshot is not finalized. There are a few more things I would like to add to it, and I'm sure
it will go through a few more rounds of editing and polishing. However, it gives you a good feel for the direction I am
going. I wanted to make the auto-width/height feature more discoverable so I made the width and height fields editable
combo boxes instead of just text boxes. Width and height prompts have been added for UI clarity, and the much requested
units field is present. As of right now, these are the only new features (well, one feature and a couple clarifications)
I intend to add to this dialog; all other features will be added to the Settings dialog or through Windows Shell
extensions.

The Settings dialog is where most of the new features will go. I decided to put them here because many of them I expect
will not be per-job settings, but rather one-time configurations. These will be things like the output file name format,
the list of default sizes (Small, Medium, Large, etc.), enabling/disabling the check for updates option,
enabling/disabling the ignore image rotations option, and maybe even specifying the resize algorithm and image quality.
Accessing this dialog will done from the start menu and probably from somewhere in the above dialog (still working out
the details).

All in all, I love the direction this software is going, and I can't wait to see the reception it gets from you users.
Be sure to comment or send me an email if you have something to say. Positive or negative, I always listen.


  [1]: http://imageresizer.codeplex.com
