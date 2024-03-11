---
title: Quarkus Tricks
description: Some handy tricks related to quarkus
published: true
date: 2024-03-11T18:36:37.141Z
tags: 
editor: markdown
dateCreated: 2024-03-11T18:36:37.141Z
---

# Quarkus Tricks
## Killing existing instance
this command isn't optimal by any means, if you have a better parsing logic, please let me know!
```bash
kill $(sudo ss -lptn 'sport = :8080' | tail -n +2 | cut -d '=' -f 2 | cut -d ',' -f 1) 
```