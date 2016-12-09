---
layout: post  
title:  "基于Node.js的爬虫工具 - Node Crawler"  
category: news  
author: "yinxin630"
---

作者：[碎碎酱][1]

[Node Crawler][2]的目标是成为最好的node.js爬虫工具，目前已经停止维护。

## 特性：

* 简洁的API
* 服务端DOM以及jQuery语法操作DOM（默认Cheerio实现，可以修改为JSDOM）
* 可配置的缓冲池大小以及自动失败重试
* 请求优先级
* forceUTF8模式使其可以自动进行编码检测以及转码
* 本地缓存
* 支持node.js 0.10及更高版本

## 示例代码：

我们来抓取光合新知博客tech栏目中的文章信息。
访问`http://dev.guanghe.tv/category/tech/`，右键查看页面源代码，可以看到文章信息等内容，如下所示：

```
<ul class="posts">
    <li>
        <span class="post-date">Dec 31, 2015</span>
        <a class="post-link" href="/2015/12/Getting-Started-With-React-And-JSX.html">React和JSX入门指导</a>
    </li>
    <li>
        <span class="post-date">Dec 30, 2015</span>
        <a class="post-link" href="/2015/12/ReactJS-For-Stupid-People.html">React 懒人教程</a>
    </li>
</ul>
```

因为每篇文章都是一个`<li>`标签，所以我们从页面代码的所有`<li>`中获取文章的发布时间、链接和标题。

爬虫代码：

```
var Crawler = require('crawler');

var crawler = new Crawler({
    maxConnections: 10,
    callback: function(err, result, $) {
        $('li').each(function(index, li) {
            console.log(index + ' :');
            console.log('time:' + $(li).children(0).text());
            console.log('url:' + result.uri + $(li).children(1).attr('href'));
            console.log('title:' + $(li).children(1).text());
        });
    }
});

crawler.queue('http://dev.guanghe.tv/category/tech/');
```

`npm install`安装crawler模块，`node app.js`运行程序。
你将会获得如下内容（仅展示部分内容）：

```
0 :
time:Dec 31, 2015
url:http://dev.guanghe.tv/category/tech//2015/12/Getting-Started-With-React-And-JSX.html
title:React和JSX入门指导
1 :
time:Dec 30, 2015
url:http://dev.guanghe.tv/category/tech//2015/12/ReactJS-For-Stupid-People.html
title:React 懒人教程
2 :
time:Dec 24, 2015
url:http://dev.guanghe.tv/category/tech//2015/12/iOSCustomProblem.html
title:iOS开发常见问题
3 :
time:Dec 17, 2015
url:http://dev.guanghe.tv/category/tech//2015/12/iOSXcodeDebug.html
title:Xcode Debug技巧
```

  [1]: http://www.suisuijiang.com
  [2]: https://github.com/sylvinus/node-crawler