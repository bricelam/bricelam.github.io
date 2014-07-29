---
layout: post
title: Image Resizer 3 CTP
permalink: /2010/10/image-resizer-3-ctp.html
tags: image-resizer
---

Lately, I've been working on version 3 of my [Image Resizer][1]. I think I'm finally narrowing in on the interface design
(your feedback is welcomed). The text may change a bit for the final release, and I might add a footer to the dialog,
but I'm pretty happy with the way it's looking.

![Image Resizer 3 CTP Screenshot]({{ site.baseurl }}/attachments/ImageResizer3Ctp.png)

The Advanced settings link will pop up another dialog box that will let you set things like the following.

* JPEG Quality Level
* Custom default sizes (edit or replace the Small, Medium, Large, etc. options)
* Resized image filename
* Disable copying of metadata
* Disable check for updates
* Disable usage reporting

After clicking Resize, it will go to a progress page while your pictures are resized. I intend to make the resize
process multi-threaded so if you have more than one CPU, you should notice some good speed improvements for those large
resize jobs.

When the resize is completed, a summary page will be displayed letting you know of any errors or warnings that occurred
during the resize.

I am about a week or two away form releasing a preview of the new version. That preview will include all the
functionality of the current version, plus a handful of the more basic features:

* Resize units (Pixels, Percent, Inches, and Centimeters)
* Resize to a specific folder
* Put replaced images in the Recycle Bin

And of course, the things that caused me to rewrite a version 3 in the first place:

* Sexy new UI
    * Progress bar
* Resize using WIC (instead of GDI+)
    * More metadata is copied
    * Any WIC driver/codec is supported
    * Animated GIFs can be resized
    * Multi-page TIFFs too
* Multi-threaded resizing

After the first CTP release, I'll continue to whittle away at the features, and I'll keep the CTPs coming until I
finally reach feature complete for version 3. Wow, I have a lot of work to do!

Oh, a few more goals that I have for version 3 that didn't get mentioned:

* Documentation
    * Screencasts
* Re-branding
* Evangalizm
    * Youtube screencasts
    * Facebook page
    * Twitter hashtag
* Translations to other languages
* A single setup (for all of 32 and 64 bit, English and non-English versions alike)


  [1]: http://imageresizer.codeplex.com
