---
layout: post
title:  "cron的坑"
category: news
author: "diggzhang"
---
cron跑定期脚本理所当然的样子,可是发现实际部署过程中,手动可以跑的脚本放在cron里歇菜.
一点点排查后发现是因为cron不会读取当前的环境变量.
***
解决方法,把下面代码加到每个待执行脚本头部:
```shell
    #!/bin/bash
    #指定bash,重新导出环境变量.
    PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
    export PATH
```
