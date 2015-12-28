---
layout: post
title:  "mongoshell extension"
category: news
author: "diggzhang"
---

mongo will read the .mongorc.js file from the home directory of the user invoking mongo. In the file, users can define variables, customize the mongo shell prompt, or update information that they would like updated every time they launch a shell.

here's an example of `~/.mongorc.js`

```
// use for query one doc only by id string
// input userId {string}
DBCollection.prototype.onedoc = function (userId) {
  return this.findOne({"_id": ObjectId(userId)});
}

//use for query id with pipeline
//input userId {string}, pipeline {json object}
DBCollection.prototype.fid = function (userId, pipeline) {
  return this.find({"_id": ObjectId(userId)}, pipeline);
}

// opponent of db.collections.findOne()
DBCollection.prototype.findLast = function (pipeline) {
  return this.find(pipeline).sort({"_id":-1}).limit(1).pretty();
}
```

shutUp && zhuangBility:

```
// findOne with out 'ObjectId()'
mongoshell> db.collection.onedoc("5680zzzzzzzzzzzzzzzzzzfe")

// findOne's brother
mongoshell> db.collection.findLast()

// just input string of ObjectId with pipeline
mongoshell> db.collection.fid("5680zzzzzzzzzzzzzzzzzzfe"), {"name": 1})
```
