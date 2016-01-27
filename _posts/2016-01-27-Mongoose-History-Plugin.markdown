---
layout: post  
title:  "Mongoose History Plugin"  
category: news  
author: "libook"
---

# [项目主页](https://github.com/nassor/mongoose-history)

这是个Mongoose的插件，可以自动地帮你记录某Collection中的所有Document的所有历史快照，
当这个Document被插入、更新或删除的时候都会在另一个Collection里生成一条历史记录，记录的Schema如下：

```javascript
{
    _id:  ObjectId,
    t: Date // 这一条历史的创建时间
    o: "i" (插入) | "u" (更新) | "r" (删除) // Document所受到的操作
    d: {  // 操作后的Document
    }
}
```

在某中场合下真的是很实用的插件啊。

我粗略的看了一下源代码，里面有写如何在调用Model.save()的时候判断是插入操作还是更新操作，有兴趣的朋友可以去看看源代码。
