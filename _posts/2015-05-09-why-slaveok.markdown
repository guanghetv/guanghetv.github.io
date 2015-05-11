---
layout: post
title:  "为什么使用slaveOK()"
category: news
author: "diggzhang"
---

如过使用副本集且mongos为1.7.4或更高版本,把读操作分散从机上,便会遇到"not master"错误.

```
> show dbs;
> Error: listDatabases failed:{ "note" : "from execCommand", "ok" : 0, "errmsg" : "not master" }

> rs.slaveOk();
```
原因是这样的,从slave机读取,意味着必须接受过时的数据,mongos无论哪种驱动都要设置"slave okay",这是为了**确认接受有可能过时的数据**.

