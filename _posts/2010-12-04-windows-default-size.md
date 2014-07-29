---
layout: post
title: A Window's Default Size
permalink: /2010/12/windows-default-size.html
---

One of my biggest pet peeves is when, after switching resolutions, your window sizes become too small to work with. This
is particularly bothersome in Media Player, Outlook, and Visual Studio. So, I wrote the following registry script to
reset them to their default sizes.

{% gist bricelam/1d7111e2ac7b882ea962 %}

Basically, it just deletes the saved window sizes, so next time you launch the application, it will be how it was the
first time you ran it. If you want to use the script, save it to a .reg file and run it. You can also open up regedit
and manually delete the values if you want.
