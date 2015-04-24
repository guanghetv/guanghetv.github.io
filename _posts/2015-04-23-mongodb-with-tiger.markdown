---
layout: post
title:  "MongoDB with WiredTiger"
category: news
author: "diggzhang"
---

###MongoDB 3.0相较于2.6而言有哪些增强?
* 写性能提升 7x-10x
* 压缩比降低30%-80%(实际测验中@libook导入8G数据后压缩到2G :D)
* 运维成本降低95%

###MMAP vs WiredTiger
MongoDB 3.0默认支持两种存储引擎:
* MMAPv1 (以下简称"MM")
* WiredTiger(only in 64-bit version, 以下简称"WT")
WT支持MM引擎所有特性,但会改变磁盘存储策略,所以需要mongodump原库后,在新的目录做mongorestore操作.有意思的是,在复制集和分片中WT和MM相互支持.

###YCSB测试
[YCSB](https://github.com/brianfrankcooper/YCSB)意思是Yahoo!Cloud System Benchmark,该工具可以对各类NoSQL进行性能测试.MongoDB3.0以YCSB测试作官方测试报告.
在YSCB测试中,Mongo 3.0在多线程批量插入场景下相较于Mongo2.6大约有7倍增长.

哦,对了,WiredTiger如果翻译成中文该叫什么呢?奇虎?
