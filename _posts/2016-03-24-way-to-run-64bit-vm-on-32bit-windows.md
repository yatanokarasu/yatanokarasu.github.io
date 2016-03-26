---
layout:         post
title:          "The way to run 64bit VM on 32bit Host OS"
description:    "If you want to run 64bit VM on VirtualBox running on 32bit Host OS, you have to enable Long mode."
keywords:       [VirtualBox, Long mode]
categories:     [Virtualization]
tags:           [Docker, VirtualBox]
---

Basically, Docker should be run on 64bit Linux distribution.
If you use 64bit Windows, you have to run Docker on virtual machine like VirtualBox.

However, we can also run Docker on VirtualBox running on 32bit Windows.
The following command allows us to use Docker on VirtualBox running 32bit Windows:

~~~
> vboxmanage.exe modifyvm ${vm_name or UUID} --longmode on
~~~

