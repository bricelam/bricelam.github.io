---
layout: post
title: "Project Idea: Windows Phone Team Explorer"
permalink: /2010/11/project-idea-windows-phone-team.html
tags: miscellaneous
---

I recently got a Windows Phone, and I had a great idea for a project: a lightweight Team Foundation Server client. It
doesn't need to be a full-blown client because there are only a handful of things that I would actually care about on a
mobile device. My initial thoughts are to prioritize the features as follows.

* Work items
    * Read and comment (Pri 0)
    * Update (Pri 1)
    * Create (Pri 2)
* Source code
    * Changesets (Pri 0)
    * Files (Pri 3)
* Build notifications (Pri 1)

The first screen would be a list of projects. I could add a project to this list by typing in the server URL, the
project name, and my credentials. After selecting a project, I could see the recently updated work items, the latest
changesets, and builds. The ability to query (and use stored queries) for work items would also be needed. Viewing the
source code files, I think, is extremely low priority as this would be a bit unwieldy on a phone. Personally, I could
also do without the build notifications since I'd mostly just use this against the CodePlex servers which don't support
that, but I do think it would be important to others.

I did a bit of prototyping on this, but didn't get very far. The Team Foundation client API, as expected, doesn't work
on the phone, and working directly with the SOAP services got a little overwhelming. I'll probably play around with it a
bit more, but feel free to steal my idea. (Just let me know so I can be your first customer.)
