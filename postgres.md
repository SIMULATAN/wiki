---
title: PostgreSQL
description: 
published: true
date: 2024-11-11T08:43:55.222Z
tags: 
editor: markdown
dateCreated: 2024-11-11T08:41:00.067Z
---

## Backup
### Using pg_dump
```bash
pg_dump -U myuser --host localhost > `date +dump_%F_%H-%M.sql`
```