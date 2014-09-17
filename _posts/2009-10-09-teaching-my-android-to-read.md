---
layout: post
title: Teaching My Android to Read
permalink: /2009/10/teaching-my-android-to-read.html
tags: android
---

About a month ago, I finally gave in and decided to join the smart phone revolution. After careful consideration, I
decided to go with the [T-Mobile MyTouch 3G][1] (powered by [Google Android][2]). I was attracted to the liberating UI,
the elegant system architecture, and, of course, the great SDK.

I had been racking my brain for a couple weeks trying to think of a great application to make when, all of a sudden, it
hit me. I was marveling over the [Barcode Scanner][3] application when I suddenly had the idea: What if I combine the
device's camera with [OCR][4]?

OCR is the process of turning pictures into text, and once you have raw text, the possibilities are endless. As a simple
example, imagine being able take a picture of an address on a brochure and having it immediately displayed on a map. The
same idea could also be applied to calling phone numbers, or even translating!

Another technology that could come in to play with this concept is the newly added TTS feature. Now you could have
things read out loud to you. Whether you can't afford to take your eyes off the road, can't see because you forgot your
glasses, or want to make every book a self-reading child book? No problem, just take a quick picture, and the text will
be read out loud.

The next step is to do layout analysis on the picture. This tells you things like at (x, y) on the picture, there is z
letter/word/block of text. So now you could just hold your phone over a page of text, and have all the instances of the
word *the* highlighted in yellow -- finally, that real-life Ctrl+F (find) you've always wanted!

Well friends, after about two weeks of development, I finally have a working prototype! I stole about 98% of the code
from the [ZXing][3] [Ocrad][5], and [STLPort][6] ([Gears][7]'s fork) projects, but hey, this is open source, and that's
how we roll. Here's how it works:

1. Hold your device over a document. The camera will auto-focus and send the image to Ocrad for processing.
2. Once Ocrad has identified some text, it will be returned and displayed on the screen.

Here is an example of it working on the Domino's ad that was on my door this afternoon:

![Large 3-Topping Pizza]({{ site.baseurl }}/attachments/Literate.jpg)

Surprisingly, it only took Ocrad took about 200ms to process the entire image on the device. It takes Barcode Sanner
about twice as long to process a 2D barcode. Although, I did cheat by using a native processor (Barcode Scanner has the
overhead of Java), and mine only has to scan for one type of image (Barcode Scanner scans for many different formats for
each picture).

Next, I'm going to get it processing the layout and visually overlaying it somehow on the screen. I have a feeling
though that this is going to add a pretty big hit to the performance, but as long as I can keep it at least as good as
Barcode Scanner, I think it will be acceptable.

When I have time, I'll also try to get the code posted on [CodePlex][8] for those interested in seeing the exact details
of how it was all done.


  [1]: http://www.t-mobilemytouch.com
  [2]: http://www.android.com
  [3]: http://code.google.com/p/zxing
  [4]: http://en.wikipedia.org/wiki/Optical_character_recognition
  [5]: http://www.gnu.org/software/ocrad
  [6]: http://stlport.sourceforge.net
  [7]: http://code.google.com/p/gears
  [8]: http://www.codeplex.com
