---
title: FFMPEG
description: 
published: true
date: 2025-02-02T12:25:44.633Z
tags: 
editor: markdown
dateCreated: 2025-02-02T12:25:44.633Z
---

# Audio
Convert downloaded YouTube soundeffect to OGG + crop + quality + strip metadat
```bash
# `ss` and `to` format: `ss:mm
ffmpeg -ss 0.4 -to 3.2 -i file.webm -map_metadata -1 -q 1 -map a sound.ogg
```