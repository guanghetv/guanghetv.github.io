---
layout: post  
title:  "Koa Pagination"  
category: news  
author: "libook"
---

# [Koa Pagination 项目主页](https://github.com/seegno/koa-pagination)

这是个Koa的中间件，可以自动生成遵循HTTP标准的利用Request头Range字段和Response头Content-Range字段来实现分页机制。

具体用法请参照项目GitHub主页的README，我这里稍稍讲一下HTTP标准里的两个头字段Range和Content-Range的用法：

假设我要做一个API，这个API是要到数据库里查所有的条目（假设总共124条），结果以时间递减排列，并且按照分页配置输出。

请求头中添加一个Range字段，格式为：

```
Range: item=<start index>-<end index>
```

比如从索引为5到10的元素（注意索引是从0开始的）那么就是：

```
Range: item=5-10
```

那么使用这个中间件返回的数据就应该是所有元素排序后的第6-11个元素，
返回的头中有一个Content-Range字段，格式为：

```
Content-Range: item <start index>-<end index>/<total number>
```

那么对应上面的请求就应该是：

```
Content-Range: item 5-10/124
```