---
title: HomeAssistant
description: 
published: true
date: 2025-02-08T16:35:16.717Z
tags: 
editor: markdown
dateCreated: 2025-02-08T16:35:16.717Z
---

# Using SQL
https://github.com/expaso/hassos-addon-timescaledb

## Queries
### Deleting states by entity id
```sql
DELETE FROM states
WHERE metadata_id IN
	(SELECT metadata_id FROM states_meta WHERE entity_id = 'sensor.my_unnecessary_sensor');
```