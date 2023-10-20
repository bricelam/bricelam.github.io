---
layout: post
title: Command Prompt is still kinda ugly
tags: miscellaneous
---

Nearly a decade ago, I wrote about how [Command Prompt was ugly]({% post_url 2014-08-14-command-prompt-is-ugly %}). We've come a long way since then. [Windows Terminal](https://aka.ms/terminal) is infinitely more beautiful, but I still don't like the default color scheme.

The first time I remember thinking, "Wow, this terminal is beautiful!" was twenty years ago when I first installed [Gentoo Linux](https://www.gentoo.org/). It was a high-resolution (well, for the time), framebuffered terminal, and something about the blue and green hues immediately sparked joy.

Years later, researched it, and it turned out that there wasn't anything special about Gentoo's color scheme. It's just the default [VGA text mode](https://en.wikipedia.org/wiki/VGA_text_mode) palette. But, it was so much more beautiful than the vintage Windows palette was.

Vintage                                                            | VGA
:-----------------------------------------------------------------:|:---:
![Vintage scheme]({{ "/attachments/Vintage.png" | absolute_url }}) | ![VGA scheme]({{ "/attachments/VGA.png" | absolute_url }})

Those bright colors just look so much better to me!

Another thing I love about the VGA pallet is the orange they used instead of that ugly olive color. *None* of the preset color schemes in Windows Terminal have duplicated this improvement. All the dark yellow hues are still kinda ugly. What a shame!

| Campbell                                                             |
|:--------------------------------------------------------------------:|
| ![Campbell scheme]({{ "/attachments/Campbell.png" | absolute_url }}) |

There is one thing, however, that the Windows Terminal schemes do better. Some of 'em use a lovely shade of purple for their dark magenta hue. So, in order to create the perfect color scheme, I mathematically reverse engineered the formula the VGA palette uses to shift dark yellow to orange, and I used it to shift the dark magenta to purple. Behold!

| Brice's Color Scheme                                                        |
|:---------------------------------------------------------------------------:|
| ![Brice's scheme]({{ "/attachments/BriceColorScheme.png" | absolute_url }}) |

Oh, I also created this PowerShell script you can use to install it.

```powershell
$settingsPath = "$env:LOCALAPPDATA\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json"
$settings = ConvertFrom-Json (Get-Content -Raw $settingsPath)
$settings.schemes += [PSCustomObject]@{
    name                = "Brice's Color Scheme"
    black               = '#000000'
    blue                = '#0000AA'
    green               = '#00AA00'
    cyan                = '#00AAAA'
    red                 = '#AA0000'
    purple              = '#5500AA'
    yellow              = '#AA5500'
    white               = '#AAAAAA'
    brightBlack         = '#555555'
    brightBlue          = '#5555FF'
    brightGreen         = '#55FF55'
    brightCyan          = '#55FFFF'
    brightRed           = '#FF5555'
    brightPurple        = '#FF55FF'
    brightYellow        = '#FFFF55'
    brightWhite         = '#FFFFFF'
    foreground          = '#AAAAAA'
    background          = '#000000'
    cursorColor         = '#FFFFFF'
    selectionBackground = '#FFFFFF'
}
Set-Content $settingsPath (ConvertTo-Json $settings -Depth 100)
```
