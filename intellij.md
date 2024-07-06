---
title: IntelliJ
description: Tips and tricks related to IntelliJ
published: true
date: 2024-07-06T08:09:03.971Z
tags: 
editor: markdown
dateCreated: 2024-07-06T08:09:03.971Z
---

# IntelliJ
## Native Wayland
From my personal testing on Hyprland, the native wayland experience is much better than whatever xwayland tries to do.

In order to enable it, go to Help -> Edit Custom VM Options and add this line:
```
-Dawt.toolkit.name=WLToolkit
``` 

In order to use the WLToolkit, you need the newest build of the JBR (JetBrains Runtime, a Java fork / distribution for JetBrains IDEs). Download it from here: https://github.com/JetBrains/JetBrainsRuntime/releases
(you'll want a jbr 21 launch release, for example `linux-x64 	JBR with JCEF (bundled by default)`)
Then, extract it somewhere and set this environment variable:
```
IDEA_SDK=/path/to/jbr
```
(WITHOUT `bin`, make sure the variable is set system-wide and not just in your shell to enable launching through 3rd party applications like rofi)