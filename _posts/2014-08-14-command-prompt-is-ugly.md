---
layout: post
title: Command Prompt is Ugly
---

As a developer, I spend a fair amount of time in Command Prompt. Let's face it, it's ugly.

![Command Prompt]({{ "/attachments/CommandPromptOriginal.png" | absolute_url }})

Luckily, all of the font, colors, and sizes are configurable. I use the following theme which, I think, looks
significantly better.

![Themed Command Prompt]({{ "/attachments/CommandPromptThemed.png" | absolute_url }})

Here are the registration entries you can use to apply it.

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Console]
"ColorTable00"=dword:00000000
"ColorTable01"=dword:00aa0000
"ColorTable02"=dword:0000aa00
"ColorTable03"=dword:00aaaa00
"ColorTable04"=dword:000000aa
"ColorTable05"=dword:00aa00aa
"ColorTable06"=dword:000055aa
"ColorTable07"=dword:00aaaaaa
"ColorTable08"=dword:00555555
"ColorTable09"=dword:00ff5555
"ColorTable10"=dword:0055ff55
"ColorTable11"=dword:00ffff55
"ColorTable12"=dword:005555ff
"ColorTable13"=dword:00ff55ff
"ColorTable14"=dword:0055ffff
"ColorTable15"=dword:00ffffff
"CurrentPage"=dword:00000004
"FaceName"="Consolas"
"FontFamily"=dword:00000036
"FontSize"=dword:00100000
"FontWeight"=dword:00000190
```

Save them to a .reg file and double-click it.
