---
layout: post
title:  "MongoDB 3.0 复制集升级操作以及坑"
category: tech
author: "diggzhang"
---

目前我们的MongoDB复制集,模式大概如下:

```
Mongo-Pri => Mongo-Sec (Master => Slave)
```

###滚动升级
MongoDB的升级非常简单,大概遵循以下步骤

```
添加repo到/etc/yum.repos.d/mongodb-org-3.0.repo
#yum update
#yum install -y mongodb-org
```

复制集升级则需要主从滚动更换,遵循以下步骤逐个升级

```
先安全停掉Primary,权重高的slave将会变成master
> rs.stepDown()
> rs.status()  //查看状态
// 然后开始升级操作,升级完成后重新开启mongod进程
```

###复制集操作"Not Master"
在复制集升级到MongoDB 3.0后,当我们访问数据库时,执行查询会发现"not master"错误:
```
> show dbs;
> Error: listDatabases failed:{ "note" : "from execCommand", "ok" : 0, "errmsg" : "not master" }

```
解决方法,在mongoshell里执行:

```
SECONDARY> rs.slaveOk()
```
然后便可以进行正常操作.

###Mongoose
Mongoose Schema对复制集进行操作时,也会遇到"not master",有两种解决方法:

- schema后添加{read: 'secondaryPreferred'}, 如果是完全面向复制集的操作,推荐此方法

```
var testSchema = new Schema({
    icon: String,
    desc: String,
    guideVideo: { type: ObjectId, ref: 'Video' }
}, { read: 'secondaryPreferred'} );
```
 
- 涉及查询的代码中执行read().exec(), 在read()中声明**read('secondary')**

```
   Point.distinct('user', queryObj)
        .read('secondary')
        .exec(function (err, result) {
            if (err) {
                console.error(err.stack);
                return callBack(err);
            }
            return callBack(null, result);
        });
```

参考链接:
[mongoose issuse2927](https://github.com/Automattic/mongoose/issues/2927#issuecomment-96996609)
