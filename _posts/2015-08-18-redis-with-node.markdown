---
layout: post
title:  "redis with node"
category: news
author: "diggzhang"
---

Redis的存储模型(K/V)：

```
    Key ---> value
```

Redis默认的操作都是在0号数据库内进行。（Redis默认有16个数据库，编号从0到15号）Redis的每个数据库都是独立运行的。我们在对Redis做读写操作的时候，应该指定主机及端口号。

```
    $ redis-server -h localhost -p 2333
```

npm包`node_redis`:

```
    $ npm install redis
```
