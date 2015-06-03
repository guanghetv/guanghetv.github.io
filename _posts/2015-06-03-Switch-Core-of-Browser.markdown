---
layout: post
title:  "自动切换浏览器核心"
category: tech
author: "libook"
---


[原文](http://www.qoophp.com/archives/639)
在页面中插入如下meta标签，可以使得用户使用360、搜狗等双内核浏览器的时候自动使用极速内核（WebKit内核）。

```html
<meta name="renderer" content="webkit"><!--用在360中-->
<meta name="force-rendering" content="webkit"><!--用在其它-->
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1"/>
```

顺序不可以变，否则360浏览器不会成功切换到极速的内核。

这里还有一个内容类似的[文章](http://clin003.com/chrome/360meta-renderer-2945/)。

