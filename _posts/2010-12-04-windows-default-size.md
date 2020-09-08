---
layout: post
title: A Window's Default Size
permalink: /2010/12/windows-default-size.html
---

One of my biggest pet peeves is when, after switching resolutions, your window sizes become too small to work with. This
is particularly bothersome in Media Player, Outlook, and Visual Studio. So, I wrote the following registry script to
reset them to their default sizes.

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\MediaPlayer\Preferences]
"XV11"=-
"YV11"=-
"WidthV11"=-
"HeightV11"=-

[HKEY_CURRENT_USER\Software\Microsoft\Office\15.0\Outlook\Office Explorer]
"Frame"=-

[HKEY_CURRENT_USER\Software\Microsoft\VisualStudio\12.0]
"MainWindow"=-
```

Basically, it just deletes the saved window sizes, so next time you launch the application, it will be how it was the
first time you ran it. If you want to use the script, save it to a .reg file and run it. You can also open up regedit
and manually delete the values if you want.
