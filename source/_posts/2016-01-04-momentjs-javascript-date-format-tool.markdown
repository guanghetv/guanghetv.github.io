---
layout: post  
title:  "Node.js时间格式化工具 - Moment.js"  
category: news  
author: "yinxin630"
---

作者：[碎碎酱][1]

[Moment.js][2] - JavaScript的时间解析、校验、操作、展示工具。

## 常用格式：

* 年：`YYYY(2016)|YY(16)|GGGG(2016)|GG(16)|gggg(2016)|gg(16)`
* 月：`M(1)|MM(01)|MMM(Jan)|MMMM(January)`
* 日：`D(1)|DD(01)|Do(1st)`
* 时：`h(1)|hh(01)|H(13)|HH(13)`
* 分：`m(1)|mm(01)`
* 秒：`s(1)|ss(01)`
* 毫秒：`S(1)|SS(01)|SSS(001)`
* 上下午：`a(am)|A(AM)`
* 星期：`dd(mon)|ddd(monday)|e(1)|E(1)`
* 周数：`W(1)|WW(01)`
* 天数：`DDD(1)|DDDD(001)`

## 测试用例：
```
var moment = require('moment');

console.log(moment().toString()); //Mon Jan 04 2016 15:20:39 GMT+0800
console.log(moment().format('GGGG-MM-DD hh:mm:ss A')); //2016-01-04 03:20:39 PM
console.log(moment().format('MMM Do, GGGG')); //Jan 4th, 2016
console.log(moment().format('gggg, W, D')); //2016, 1, 4
console.log(moment('2015, January 4th', 'YYYY, MMMM Do').toString()); //Sun Jan 04 2015 00:00:00 GMT+0800
console.log(moment('2012-12-12', 'YYYY-MM-DD').toString()); //Wed Dec 12 2012 00:00:00 GMT+0800
console.log(moment('2013/11/14 08:12:45', 'YYYY/MM/DD h:m:s').toString()); //Wed Nov 14 2012 08:12:45 GMT+0800
```


  [1]: http://www.suisuijiang.com
  [2]: http://momentjs.com/docs/#/parsing/string-formats/