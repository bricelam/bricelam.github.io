---
layout: post
title: Image Resizer 3.0 Brainstorming
permalink: /2010/01/image-resizer-30-brainstorming.html
tags: image-resizer
---

Today I did a little brainstorming about what I want version 3 of the [Image Resizer Powertoy Clone for Windows][1] to
be like. Probably the biggest thing I want to do is a complete overhaul of the user interface. This will probably
include things like:

* More entry points
    * Explorer toolbar resize option
    * Drag-and-drop menu handler
* Better error reporting
* Branding

Just so you can get a feel of what I'm talking about, here is a little prototype I made today:

![Resize Pictures Prototype]({{ "/attachments/ResizePictures3.png" | prepend: site.baseurl | prepend: site.url }})

Another thing I want to do is rip out all the GDI+ code and replace it with the newer Windows Imaging Component (WIC)
technology. WIC handles a lot more metadata, and it is extensible so my program could in theory support every image
format known to man (assuming someone develops a WIC codec for it).

Last, but not least, I would like to address some of the feature requests that have been out there for a while. For
example:

* Check for updates
* One installer for both 32-bit and 64-bit versions.
* Settings dialog
    * Specify default sizes
    * Specify resized file name format

Who knows, maybe I'll finally get some end-user documentation to put out there too. The future has much in store for
this little utility!


  [1]: http://imageresizer.codeplex.com
