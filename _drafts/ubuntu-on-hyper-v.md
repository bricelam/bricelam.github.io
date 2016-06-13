---
layout: post
title: Ubuntu with Hyper-V
---

Getting Ubuntu running at its best on Hyper-V is not an obvious task. This post is a compilation of my findings. If you
have any other tips, please let me know!

New Virtual Machine...
======================

In the New Virtual Machine Wizard, choose to create **Generation 2** virtual machine.

![Generation 2 Virtual Machine]({{ site.baseurl }}/attachments/HyperVGeneration2.png)

Set the **Startup memory** to **1024**. Check **Use Dynamic Memory for this virtual machine.**

Settings
========

Before starting your virtual machine, edit the following settings.

On the Firmware page, uncheck **Enable Secure Boot**

On the Processor page, crank the **Number of virtual processors** up as high as it will go.

Grub
====

```shell
nano /etc/default/grub
```

```conf
GRUB_CMDLINE_LINUX="video=hyperv_fb:1920x1280 elevator=noop"
```

If you have more than 7 virtual processors or over 30GB RAM, also add `numa=off`.

```shell
update-grub
```

Virtual machine tools
=====================
```shell
sudo apt-get install --install-recommends linux-tools-virtual-lts-xenial linux-cloud-tools-virtual-lts-xenial
```