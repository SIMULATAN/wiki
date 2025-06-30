---
title: PostgreSQL
description: 
published: true
date: 2025-06-30T16:01:06.687Z
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

## Disk Space
### Links
- https://wiki.postgresql.org/wiki/Disk_Usage
- https://www.sqlbot.co/blog/postgres-table-size-how-to-debug-your-db-to-find-the-biggest-tables-rows-and-cells
- https://www.postgresql.org/docs/current/routine-vacuuming.html#VACUUM-FOR-SPACE-RECOVERY
  - https://www.postgresql.org/docs/current/sql-vacuum.html
  
### Analyze TOAST size of table
```sql
SELECT 
  pg_size_pretty(pg_total_relation_size('pg_toast.pg_toast_<id>')) AS toast_size,
  pg_size_pretty(pg_relation_size('pg_toast.pg_toast_<id>')) AS toast_table,
  pg_size_pretty(pg_indexes_size('pg_toast.pg_toast_<id>')) AS toast_indexes;
```

### Largest Tables
#### Compact
```sql
SELECT                                         
    relname as table,
    PG_SIZE_PRETTY(PG_TOTAL_RELATION_SIZE(relid)) as total_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC,
         pg_relation_size(relid) DESC
LIMIT 10;
```

#### Expressive (with TOAST size)
```sql
SELECT *, pg_size_pretty(total_bytes) AS total
    , pg_size_pretty(index_bytes) AS index
    , pg_size_pretty(toast_bytes) AS toast
    , pg_size_pretty(table_bytes) AS table
  FROM (
  SELECT *, total_bytes-index_bytes-coalesce(toast_bytes,0) AS table_bytes FROM (
      SELECT c.oid,nspname AS table_schema, relname AS table_name
              , c.reltuples AS row_estimate
              , pg_total_relation_size(c.oid) AS total_bytes
              , pg_indexes_size(c.oid) AS index_bytes
              , pg_total_relation_size(reltoastrelid) AS toast_bytes
          FROM pg_class c
          LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
          WHERE relkind = 'r'
  ) a
) a;
```

### Largest Rows
```sql
SELECT 
        r.id as row_id,
        PG_SIZE_PRETTY(sum(PG_COLUMN_SIZE(r.*))) as row_size
FROM <table> as r
GROUP BY r.id
ORDER BY sum(pg_column_size(r.*)) DESC
LIMIT 10;
``` 