---
layout: post
title: Android + Rhino = ?
permalink: /2009/11/android-rhino.html
tags: android
---

So, what do you get when you cross an android with a rhinoceros? I have no idea, but that's exactly what I've been doing
over the past few weeks.

I recently downloaded the [Android Scripting Environment][1]> (ASE) that I discovered on [Google Labs][2]. ASE is a
nifty little project that allows you to write scripts for your Android device. At the time, the project supported the
following languages:

* Lua
* BeanShell
* Python
* Ruby (via JRuby)
* Perl

The problem is that I don't really know any of those languages. So, I started learning a little bit about each of them.
I finally decided to go with Ruby since it was the most object-oriented of the bunch, but about half way through the
tutorials, I had an epiphany: "What about JavaScript? Everybody knows JavaScript." So I decided to submit a feature
request.

I navigated my world-wide web browser over to the project's issue tracker, and being the good user that I am, did a
quick search for the word *JavaScript*. The query brought back a single, five-month-old issue titled [Add JavaScript
support][3]. I read through it and noticed that they suggested doing it using an open source Java implementation of
JavaScript by Mozilla called [Rhino][4]. I didn't think much about it at the time, I just starred the issue (i.e. added
my vote to implement it) and went on with my day.

Later that day, I got to thinking: "You know, that issue is five months old. They've already implemented five different
interpreters. I may not know anything about Rhino or ASE, but I'm a pretty capable developer and could probably get it
working in a matter of days if I really tried." So, I asked them if they'd accept a user-contributed patch. They said
they'd be glad to. I got to work on making it happen.

Implementing it was pretty straight forward. On the ASE side, I just needed to add a couple classes - one telling it
about the new interpreter, and the other telling it how to start the interpreter. For the Rhino side, I just compiled
the jar with debugging off (to save storage space) and ran it through the [dx tool][5] (so Android could understand the
compiled code). The dx tool took me a little while to figure out because I kept getting an out of memory error when
running it, but once I learned that the default stack size was too small, things fell into place fairly quickly. Now,
the only other thing I had to do was create a JavaScript proxy class that scripts could use to make [JSON-RPC][6] calls
to the Android API (via the [AndroidFacade][7]). I followed suite with the BeanShell and Ruby implementations and was
able to come up with a fairly elegant solution. Now everything was in place, and it was time test.

I immediately started getting errors the first time I went to test it. Using the interactive shell, I was able to call
methods through the proxy class but wouldn't get any return values, and the executing saved scripts just spewed an ugly
stack trace all over the screen. The first problem (no return values) was caused by an ambiguity in the `JSON.parse()`
method to Rhino so, I figured the string was coming form a fairly trusted source and just went with JavaScript's
built-in `eval()` method instead. The next problem was not so straight forward. Investigating the ugly stack trace
showed that it was yet another out-of-memory error. I fixed that problem (again by increasing the stack size), but yet
another error appeared: *Class format not accepted*. This threw me off. At first, I thought Dalvik (Android's JVM) was
trying to load the JavaScript file directly, but that didn't make sense since other interpreters were working fine.
After a few days of meditating and some ritual sacrifices to the programming gods, it hit me: Rhino was compiling the
JavaScript file to a Java class file and then handing that class file to Dalvik which had no idea what to do with it
(since it only understands Dex files). So I turned off Rhino's optimization (thus causing it to interpret and not
compile the scripts), and all was well. The Rhinodroid was complete!

I contributed my code, and waited. It took a few days, but [Damon Kohle][8] reviewed it and [merged it into the main
codebase][9]. About the only change I've noticed was that he set the default buffer sizes (making a couple of debug
messages go away). He was really quick to get [a new release][10] out so now, thanks to my obsession with coding, users
can now choose from JavaScript along with the other languages I mentioned previously.

All in all, this was a very fun experience. I got to learn more about Android and Rhino development, and I even got to
use a distributed version control system for the first time. JavaScript is probably the widest known scripting language,
and I hope that my contribution can help add to the success of ASE and can be yet another of my little claim-to-fames.
But most of all, I'm proud that I can finally say, "I've contributed to an open source Google project!"


  [1]: http://code.google.com/p/android-scripting
  [2]: http://www.googlelabs.com
  [3]: http://code.google.com/p/android-scripting/issues/detail?id=4
  [4]: http://www.mozilla.org/rhino
  [5]: http://developer.android.com/guide/developing/tools/othertools.html#dx
  [6]: http://json-rpc.org
  [7]: http://code.google.com/p/android-scripting/wiki/AndroidFacadeAPI
  [8]: http://code.google.com/u/damonkohler
  [9]: http://code.google.com/p/android-scripting/source/detail?r=835425b4ac473f2a2acfe9c4c2b92343f31b36ec
  [10]: http://code.google.com/p/android-scripting/downloads/detail?name=ase_r14.apk
