---
title: Snoty
description: 
published: true
date: 2024-10-28T14:21:02.308Z
tags: 
editor: markdown
dateCreated: 2024-10-28T14:21:02.308Z
---

# Snoty
## Database
### Migrate user data
```js
const oldUUID = "..."
const newUUID = "..."
db.getCollectionNames()
  .filter(name => !name.startsWith("jobrunr:"))
  .map(c => db.getCollection(c))
  .forEach(collection => collection.updateMany({userId: UUID(oldUUID)}, {userId: {$set: UUID(newUUID)}}))
```