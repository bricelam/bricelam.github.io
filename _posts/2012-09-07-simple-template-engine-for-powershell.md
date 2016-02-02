---
layout: post
title: Simple Template Engine for PowerShell
permalink: /2012/09/simple-template-engine-for-powershell.html
tags: powershell
---

Here is a simple, token-replacement template engine for PowerShell that you might find useful.

{% gist bricelam/a5debdbfc495eb7b116c %}

You use it like this.

```powershell
Merge-Tokens 'Hello, $target$! My name is $self$.' @{
        Target = 'World'
        Self = 'Brice'
    }
```

The output of that command, as you would expect, is this.

    Hello, World! My name is Brice.

Enjoy.
