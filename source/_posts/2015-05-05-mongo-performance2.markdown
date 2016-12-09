---
layout: post
title:  "mongoDB性能优化"
category: news
author: "diggzhang"
---

###查询性能优化Tips:

- 涉及排序操作使用索引.
- MongoDB使用Btree索引,涉及到范围的索引,应该尽可能放在符合查询逻辑的最后面.

有没有想过,排除本身SQL语句问题,为什么读操作耗时?这个问题还是要从mongo的查询机制说起,查询内容不在RAM中,mongo会放在虚拟内存中,虽然给给该资源一个指针,但该指针是指向硬盘的.

###黑魔法 Moving Databases into RAM

此黑魔法仅适合于内存容量足够承载数据库的场景,启动mongod后,系统会知道数据库在内存中,更快的读取资源.

```
$ for file in /data/db/dbname.*
> do
> dd if=$file of=/dev/null
> done
```
