---
layout: post
title: Using WinMerge with TortoiseGit
tags: git
---

I love [WinMerge][1]. My favorite features of it are syntax highlighting and moved block detection. I also love
[TortoiseGit][2]. Let's look at how to use WinMerge with TortoiseGit.

First, open the TortoiseGit Settings dialog. On the left, select **Diff Viewer**.

![TortoiseGit Diff Viewer Settings]({{ site.baseurl }}/attachments/TortoiseGitDiffViewer.png)

Under **Configure the program used for comparing different revisions of files**, select **External** and type the
following into the box.

    WinMergeU /r /e /x /u /wl %base %mine

Next, select **Merge Tool** (on the left). Again, select **External** and type the following.

    WinMergeU /e /u %merged

Now, wherever TortoiseMerge use to show up, you should see WinMerge instead.


  [1]: http://winmerge.org
  [2]: https://code.google.com/p/tortoisegit
