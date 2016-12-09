---
layout: post
title:  "How to use Pymongo"
category: tech 
author: "diggzhang"
---


使用pymongo写mongodb的查询脚本，可以同时操作多个库，使用python方便的操作文档的数据结构，遇到一些耗时较长的查询可以不必担心超时。

###引入pymongo

```
>>> import pymongo
```

###MongoClient
使用mongoclient可以与服务器建立连接。

```
>>> from pymongo import MongoClient
>>> client = MongoClient('localhost', 27017)
或者也可以酱紫
>>> client = MongoClient('mongodb://localhost:27017/')
```
###连接到DB

```
>>> db = client.test_database
为了避免一些数据库命名带有符号，推荐使用这种方法来连接：
>>> db = client['test-users']
```
###Collection

紧接着上面，连接到数据库后，选中其中的一个collection：

```
>>> collection = db['users-collection']
```
即便是`users-collection`这个collection不存在，pymongo也会连接成功。当第一条记录insert进去之后，这个表会自动创建。

###Documents操作

```
>>> import datetime
>>> post = {"author":"Mike",
>>> 		  "text":"First Blog Post",
>>> 		  "tags":["mongodb","python","pymongo"],
>>> 		  "date": datetime.datetime.utcnow()
>>> 	    }
```
上述代码中，日期处理使用的是datetime，pymongo会将datetime的数据类型处理为mongo的时间数据类型。

插入操作：

```
>>> posts = db.posts
>>> post_id = posts.insert_one(post).inserted_id
>>> post_id
ObjectId('...')
```
获取一条记录find_one():

```
>>> posts.find_one()
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike...
```

###Querying By ObjectId

如果查询中，需要用到ObjectId，直接手工拼一个Object的string是不行的，需要用到object包，转换成真正的Object数据类型。用法简单：

```
	from bson.objectid import ObjectId
	def get(post_id):
		document = client.db.collection.find_one({"_id":"ObjectId(post_id)"})
```

###批量插入操作
3.0数据迁移必备 —— **insert_many()**, 增量式批量插入。

```
>>> new_posts = [{"author": "Mike",
...               "text": "Another post!",
...               "tags": ["bulk", "insert"],
...               "date": datetime.datetime(2009, 11, 12, 11, 14)},
...              {"author": "Eliot",
...               "title": "MongoDB is fun",
...               "text": "and pretty easy too!",
...               "date": datetime.datetime(2009, 11, 10, 10, 45)}]
>>> result = posts.insert_many(new_posts)
>>> result.inserted_ids
[ObjectId('...'), ObjectId('...')]

```

###遍历查询
pymongo的find()结合`for...in...`语法：

```
>>> for post in posts.find({"author": "Mike"}):
...   post
...
{u'date': datetime.datetime(...), u'text': u'My first blog post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'mongodb', u'python', u'pymongo']}
{u'date': datetime.datetime(2009, 11, 12, 11, 14), u'text': u'Another post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'bulk', u'insert']}

```

结合pipe语法遍历:

```
>>> d = datetime.datetime(2009, 11, 12, 12)
>>> for post in posts.find({"date": {"$lt": d}}).sort("author"):
...   print post
...
{u'date': datetime.datetime(2009, 11, 10, 10, 45), u'text': u'and pretty easy too!', u'_id': ObjectId('...'), u'author': u'Eliot', u'title': u'MongoDB is fun'}
{u'date': datetime.datetime(2009, 11, 12, 11, 14), u'text': u'Another post!', u'_id': ObjectId('...'), u'author': u'Mike', u'tags': [u'bulk', u'insert']}

```

### Counting
PV、UV什么的，难免要count()一下：

```
>>> posts.count()
3
>>> posts.find({"truthOfLife": 42}).count()
1
```

### Aggregation
有了聚合，基本可以宣布就是在写mongo查询语句了。
我们一般写个`pipe = []`，`[]`里包着查询条件。
>As python dictionaries don’t maintain order you should use SON or collections.OrderedDict where explicit ordering is required eg “$sort”

```
>>> from bson.son import SON
>>> pipeline = [
...     {"$unwind": "$tags"},
...     {"$group": {"_id": "$tags", "count": {"$sum": 1}}},
...     {"$sort": SON([("count", -1), ("_id", -1)])}
... ]
>>> list(db.things.aggregate(pipeline))
[{u'count': 3, u'_id': u'cat'}, {u'count': 2, u'_id': u'dog'}, {u'count': 1, u'_id': u'mouse'}]
```

###综合示例
以下是一个非常典型的查询脚本，了解上述基本用法后，很轻易的就能上手pymongo了。

```
$ vim countH5PagePv.py


# _*_ coding: utf8 _*_  声明编码，否则遇到汉字会报错

一堆包调用
from pymongo import MongoClient
from bson.objectid import ObjectId
import datetime

链接到数据库和表
# connect to db
dbClient = MongoClient('mongodb://localhost:27017')
db = dbClient['yangcong25']

# open collections
users = db['users']

定义了一个日期区间
startDate = datetime.datetime(2015, 10, 1)
endDate   = datetime.datetime(2015, 10, 2)

一个典型的aggregation函数调用，最终返回的是list()
def pvOfPage(startDate, endDate):
    pipeLine = [
        {"$match": {
            "createdBy": {
                "$gte": startDate,
                "$lte": endDate
            },
           "event": "enterSite"
        }}
    ]
    return list(users.aggregate(pipeLine))

pvCountFromH5 = pvOfH5Page(startDate, endDate)

用len()计算返回list的长度作为PV值
print("PV of Page : ")
print(len(pvCountFromH5))

```

