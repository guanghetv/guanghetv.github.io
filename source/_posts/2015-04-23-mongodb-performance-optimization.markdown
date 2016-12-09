---
layout: post
title:  "MongoDB Performance Optimization"
category: news
author: "diggzhang"
---

MongoDB真的遇到性能瓶颈了吗? 对于查询优化,还可以做到哪些?


##关于服务器硬件
* MongoDB使用内存映射文件I/O来访问数据存储,所以受到了OS以及内存大小的限制.64bit的Linux最大虚拟内存地址为128TB,而Windows对内存映射文件的限制为8TB(如果开启日志则只有4TB)
* 由于映射机制MongoDB会用尽RAM,不过OS会按需将内存释放给其他进程
* 通过提供合适大小的内存,MongoDB可以在内存中保持它的所需数据,由此减少磁盘I/O

##确认工作集大小
常规操作只会访问到整个库中的一部分数据,我们应该根据常用数据合理的计算硬件投入.

##官方推荐硬件标配
* RAID-10
* 云 Amazon Provision IOPS
* 复制集或备份库,应该考虑添加额外网卡形成单独网络
* 重中之重还是**RAM**

##评估查询性能
对于性能评估Mongo提供了两个工具:

* explain()   ==>  单个查询评估 
* MongoDB分析器  ==> 分析神器,它会记录统计信息以及查询执行的细节反馈到system.profile表,可以通过它发现慢查询原因.

其实我们已经发现$group是最慢的,所以应该减少group操作.另外对于不同的集合,应该调整不同的查询策略,或许换一下聚合的顺序就会大大提升查询速度.

##MongoDB分析器
* 启用分析器

```
    $mongo
     use points
     db.setProfiingLevel(1)
```

* 找到执行超过半秒的查询

```
     db.setProfilingLevel(1,500)
```

* 分析级别设置为2,对所有查询启用分析器,汇集的结果到了system.profile

```
    > db.setProfilingLevel(2)
```

* 如何分析system.profile的日志

```
    > db.system.profile.find({millis:{$gt:50}}).sort({millis:-1})
```

如上,找出了所有耗时50ms以上的查询.
