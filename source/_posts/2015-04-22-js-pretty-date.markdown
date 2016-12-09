---
layout: post
title:  "JavaScript Pretty Date"
category: news
author: "diggzhang"
---

[pretty.js](http://ejohn.org/files/pretty.js)是让日期看起来"pretty"的方式,代码仅有十几行.在jQuery中调用prettyDate()也可实现相同效果.

```javascript
    prettyDate("2015-03-28T20:24:17Z") // => "2 hours ago"
    prettyDate("2015-03-27T22:24:17Z") // => "Yesterday"
    prettyDate("2015-03-26T22:24:17Z") // => "2 days ago"
    prettyDate("2015-03-14T22:24:17Z") // => "2 weeks ago"
    prettyDate("2015-12-15T22:24:17Z") // => undefined
```

***
