---
title: Snoty
description: 
published: true
date: 2024-12-26T12:48:54.000Z
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
  .forEach(collection => collection.updateMany({userId: UUID(oldUUID)}, {$set: {userId: UUID(newUUID)}}))
```

## Backup
```bash
cd /bitnami/mongodb/data/db \
  && mkdir backup \
  && cd backup \
  && mongodump -u snoty -d snoty -p "$MONGODB_EXTRA_PASSWORDS"
```

## MongoSH
```bash
mongosh mongodb://snoty:$MONGODB_EXTRA_PASSWORDS@127.0.0.1:27017/snoty?directConnection=true
