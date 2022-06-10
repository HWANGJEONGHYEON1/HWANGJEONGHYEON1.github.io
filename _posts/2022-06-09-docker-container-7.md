---
layout: post
title:  "docker-container-7"
date:   2022-05-31 12:20:21 +0900
categories: docker
---

> 도커로 mongo, redis 실행

# MONGO

```
docker run -v ${PWD}/data:/data/db mongo

# 마운트됨
~/docker/my-mongo/data 
❯ ls
WiredTiger                           diagnostic.data
WiredTiger.lock                      index-1--5489551025360238338.wt
WiredTiger.turtle                    index-3--5489551025360238338.wt
WiredTiger.wt                        index-5--5489551025360238338.wt
WiredTigerHS.wt                      index-6--5489551025360238338.wt
_mdb_catalog.wt                      journal
collection-0--5489551025360238338.wt mongod.lock
collection-2--5489551025360238338.wt sizeStorer.wt
collection-4--5489551025360238338.wt storage.bson


# mongo exec

> use dangen
switched to db dangen
> db.dangen.insert({"name": "HWANG DANGEN"})
WriteResult({ "nInserted" : 1 })
> db.dangen.find({})
{ "_id" : ObjectId("629eed683ca98a90f18dff4e"), "name" : "HWANG DANGEN" }
>

# 컨테이너 종료

# 도커 재실행 후 재조회
> show dbs
admin   0.000GB
config  0.000GB
dangen  0.000GB
local   0.000GB

> db.dangen.insert({"name": "HWANG DANGEN"})
WriteResult({ "nInserted" : 1 })
> db.dangen.find({})
{ "_id" : ObjectId("629eed683ca98a90f18dff4e"), "name" : "HWANG DANGEN" }
{ "_id" : ObjectId("629eef11d2b38a3d96c83b20"), "name" : "HWANG DANGEN" }
>

```


# REDIS


```
docker run -p 6379:6379 redis

~/docker
❯ redis-cli
127.0.0.1:6379> set name "Hwang jh"
OK
127.0.0.1:6379> get name
"Hwang jh"
```