---
title: PostgreSQL
description: 
published: true
date: 2025-03-27T20:53:23.441Z
tags: 
editor: markdown
dateCreated: 2024-11-11T08:41:00.067Z
---

## Backup
### Using pg_dump
```bash
pg_dump -U myuser --host localhost > `date +dump_%F_%H-%M.sql`
```

## Connection Leaks
You can view all queries made by a JDBC driver using the following SQL statement. This will allow you to detect connection leaks. Look for `idle in transaction` entries and the `query` associated to find the offending part of your code.

```sql
SELECT pid
     , usename
     , application_name
     , client_addr
     , state
     , TO_CHAR(NOW() - state_change, 'HH24:MI:SS') AS state_change_diff
     , query
FROM pg_stat_activity
WHERE application_name ILIKE '%Driver%';
```

Credit: https://medium.com/@eremeykin/how-to-deal-with-hikaricp-connection-leaks-part-1-1eddc135b464