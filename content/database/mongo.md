---
title: "MongoDB"
date: 2016-12-01 07:36
---

[TOC]


## 简单使用

查询
```
db.COLLECTION_NAME.find({查询条件map}, {筛选字段map})
```

更新
```
db.COLLECTION_NAME.update({查询条件map}, {更改document})
db.COLLECTION_NAME.update({查询条件map}, {"$set": {更改字段map}})
```

插入
```
db.COLLECTION_NAME.insert({插入map})
```

删除
```
db.COLLECTION_NAME.remove({查询条件map})
```

## 索引管理

创建索引ensureIndex()
```
db.COLLECTION_NAME.ensureIndex(keys[,options])
  keys，要建立索引的参数列表。如：{KEY:1}，其中key表示字段名，1表示升序排序，也可使用使用数字-1降序。
  options，可选参数，表示建立索引的设置。可选值如下：
    background，Boolean，在后台建立索引，以便建立索引时不阻止其他数据库活动。默认值 false。
    unique，Boolean，创建唯一索引。默认值 false。
    name，String，指定索引的名称。如果未指定，MongoDB会生成一个索引字段的名称和排序顺序串联。
    dropDups，Boolean，创建唯一索引时，如果出现重复删除后续出现的相同索引，只保留第一个。
    sparse，Boolean，对文档中不存在的字段数据不启用索引。默认值是 false。
    v，index version，索引的版本号。
    weights，document，索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重

如：
  db.hq_algorithm.ensureIndex({"backtest_unique":1},{"unique":true,"sparse": true})
```

查看索引
```
db.COLLECTION_NAME.getIndexes()

查看索引大小
db.COLLECTION_NAME.totalIndexSize()
```

删除索引
```
db.COLLECTION_NAME.dropIndex("INDEX-NAME")

删除所有索引
db.COLLECTION_NAME.dropIndexes()
```


## 参考

+ [MongoDB索引管理](http://itbilu.com/database/mongo/E1tWQz4_e.html)
