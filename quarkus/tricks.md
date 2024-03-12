---
title: Quarkus Tricks
description: Some handy tricks related to quarkus
published: true
date: 2024-03-12T13:56:37.087Z
tags: 
editor: markdown
dateCreated: 2024-03-11T18:36:37.141Z
---

# Quarkus Tricks
## Killing existing instance
In case you get an error that the quarkus port is already allocated, run this to kill the process running on `8080`.
```bash
kill $(sudo ss -lptn 'sport = :8080' | tail -n +2 | cut -d '=' -f 2 | cut -d ',' -f 1) 
```
*the parsing logic is terrible, if you have an improvement, please let me know*