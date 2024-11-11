---
title: PostgreSQL
description: 
published: true
date: 2024-11-11T08:41:00.067Z
tags: 
editor: markdown
dateCreated: 2024-11-11T08:41:00.067Z
---

## Backup
### Using pg_dump
```bash
pg_dump -U myuser --host localhost > dump-`date +%F_%H`.sql
```