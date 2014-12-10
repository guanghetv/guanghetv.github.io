---
layout: post
title:  "HTTP API 设计"
categories: tech
author: "libook"
---

这是一个使用非关系型数据库实现关系模型的探索。

曾经一个月恶补MEAN框架和Rest风格，给我印象最深的就是那本翻译得堪比Google机翻的《REST实战（中文版）》，好好的中国话不能好好说，完全看不懂。
马上就接到任务设计新的班级管理系统，需要整个从数据结构开始设计，之后还要设计WEB API和客户端，没有实践就没有提升嘛，于是我硬着头皮上了。

因为这篇文章主要是讲HTTP API的设计，所以除了HTTP API设计部分以外的在这里就不赘述了。


##初试牛刀

由于此前看了好多宣传MongoDB优势的文章，现在要基于MongoDB设计数据结构还真有点小激动。

当时列出了很多方案，因为在用户看来，学校、班级和学生之间的关系是这样的：

> 学生和老师 =(属于)=> 班级 =(属于)=> 学校

学生和老师是用户，有单独的Collection，而MongoDB是面向JSON文档的，那么我完全可以把学校和班级塞入同一个Collection中，将班级的JSON文档嵌入到学校的JSON文档中——没有MongoDB使用经验的我这样想道，而且没做试验和调查就这样做了，实现的数据结构如下：

```javascript
{
    schoolId:ObjectId,
    schoolName:String,
    rooms:[
        {
            roomId:ObjectId,
            roomName:String,
            students:[ObjectId]
            teachers:[ObjectId]
        }
    ]
}
```

那时找到了一个异常强大、名字却超长的数据持久化框架——[express-restify-mongoose](http://florianholzapfel.github.io/express-restify-mongoose/)，使用这玩意儿只需要定义一个Schema，就能自动帮你创创建好所有CRUD操作的WEB API以及API对应的数据库方法，甚至支持模糊、排序、分页等高级的查询方法，开创了傻瓜写服务端的先河～～不过这东西竟然不支持嵌入式文档的查询，所以这次无缘使用了，只好自己设计WEB API和写Controller以及数据库方法。

在设计HTTP API的时候也遵照数据结构设计成了如下样式：

> /schools
> /school/:schoolId
> /school/:schoolId/rooms
> /school/:schoolId/room/:roomId
> （说明：带“:”的是参数）

乍一看很清晰呀，貌似没什么问题。

**注意！前方深坑**。

因为MongoDB查询的单位是Collection中的最顶级的Document，所以我每次查询都只会返回学校的文档，没错！即便我查询的是一个班级，它也会返回整个学校，这时我就只能用嵌套的for循环来把我真正查询的班级从学校中再“查询”出来。

显然，这个操作交给客户端去做是不合理的，那样的话如下三个WEB API的功能就一模一样了：

> /school/:schoolId
> /school/:schoolId/rooms
> /school/:schoolId/room/:roomId

所以这个工作一定是在服务器上完成的，不过大多时候我获取一个班级的信息是希望同时携带这个班级所属学校的信息的，所以我还不能直接把所需要的班级剃出来返回，我还要保持原文档的结构返回一个学校，只不过这个学校下面只有一个那个我想要的班级而已——辛勤写代码的我全然不知自己在坑中越挖越深。

这个“剃出”操作便是一段又一段的for循环操作，代码中随处可见各种嵌套的for循环，写代码的时候很痛苦，看代码的时候也很痛苦。

于是，客户端中的所谓班级模型实际上是**伪**学校模型，只不过这个学校下面只有一个班级的信息，每次使用都要一边提醒自己：“这货是个数组”，另一边把出BUG的代码做如下修改：

```javascript
//把
room.rooms.roomName;
//改成
room.rooms[0].roomName;
```

而**真**学校模型也时常存在，那么代码的可读性就被完全摧毁了，因为一段看起来极像是在处理学校模型的代码其实是在处理班级模型。。。

你以为这就结束了吗？当然没有！
有的时候我会进行班级的条件查询，那么这下拿到的不是一个包含所有符合条件的班级的数组，没错，是一个学校，而学校下面才是符合条件的所有班级。

于是很别扭地一边提醒自己：“这货是个学校对象”，另一边把出BUG的代码做如下修改：

```javascript
//把
rooms[roomIndex].roomName;
//改成
rooms.rooms[roomIndex].roomName;
```

什么？您感到一阵晕眩？哈哈，我也是@_@

写\改这些代码无疑就像是工兵排雷，一不小心就会被地雷炸死，一不小心BUG也会把服务炸死。

好不容易终于完成了编码，也好不容易跳出了测试和修BUG的循环，以为噩梦结束了，这时我的新任务也来到了，是在此基础再写一个班级管理系统，这个是面向老师的，让老师可以方便的管理自己的班级。

尽管操作的都是同一个Collection，但由于当时设计HTTP API的时候没有考虑到API的扩展性和通用性的问题，之前的这套API只能用于之前开发的系统，所以我只能再设计一套功能相似但权限和名称不同的HTTP API，于是我又经历了一遍之前的痛苦历程。。。


##含泪重构

需求是不断增加和变化的，所以在经历几次修改和再开发之后；客户端对班级模型的奇葩处理算法已经使客户端整体的代码异常冗余和混乱不堪；而服务端也由于大量的for循环而性能告急。
CTO大人慈祥地对我说：孩子，重构吧！

###反思

回顾此前的开发过程：

客户端代码冗余和混乱是由于API返回数据结构不合理；API返回数据结构不合理以及服务端性能差的共同原因，是数据本身的结构就不合理；mongoDB对数据操作的直接单位是文档(Document)，同时文档(Document)也是操作最频繁的单位（此处的“文档”有别于“嵌入式文档”，实际上MongoDB是不支持直接操作嵌入式文档的），但是在班级管理系统之中，学校显然只是一个附加信息，真正核心的操作对象其实是班级，那么班级下面有学生和老师，同时班级也有一个属性表明此班级是属于哪个学校。

对于之前的开发工作，不得不吐槽的还有：

* 由于API的controller中写了过于详细的业务逻辑（如定制的输入输出数据结构、权限判定），所以实现出来的API不具有通用性和扩展性，导致的结果是，分明是同类的功能反而要写几套名称不同的API，大大地加重了代码冗余，同时也失去了作为API应有的意义。
* HTTP API的命名规则混乱；试想一下，如果需要把同一类功能写成不同的多个API的话，API的名称是不能冲突的，就拿班级来讲，有可能不同的API我要写成如下这种：

>  /room
>  /class
>  /banJi
>  /anotherBanJi
>  ......

*  HTTP API过长；由于此前是学校里面嵌入班级的文档，那么想当然地就将API设计成了如下所示，实际上后两个API当中的school没什么意义，因为room的ID本身就是唯一的：

>  /schools
>  /school/:schoolId
>  /school/:schoolId/rooms
>  /school/:schoolId/room/:roomId

###重构内容

重构主要考虑一下四个方面：

####1. 精简

把最复杂的变成最简单的，才是最高明的。

* 精简API数量，同类功能尽量使用同一个HTTP API；
* 精简API长度，API本身在含义明显的基础上尽量简短;
* 精锐代码，算法要保持简单高效。

####2. 通用

HTTP API就像是原料，可以制作成什么东西取决于如何利用；但如果一个原料已经具有复杂的结构和功能，那么这个原料所能制成的产品就少之又少了。
为了提高功能和代码的复用率，HTTP API不需要考虑任何用户背景（如权限）以及其他冗余功能；所以很简单的，客户端发送一定的规范化的请求，经过处理后服务返回事先约定的通用数据；数据如何使用交给客户端程序来决定。
需要说明的是权限系统应该是额外的一套系统，将HTTP API放在权限系统之后，任何请求先经过权限系统的审核。

####3. 扩展

同类的请求使用同一API，对输入和输出的不同需求要么让客户端的服务自己解决，要么使用不同的API参数来定制，有前面精简和通用的铺垫，高扩展性便是自然且方便的。

####4. 合理

一栋大楼是否稳固，往往取决于地基是否牢靠。
HTTP API作为系统的基础部分，要尽可能设计地合理，特别是输入输出，一旦重构无异于将整个系统推倒重做。

###实施

解决矛盾要先解决主要主要矛盾。归根结底主要矛盾在于数据库中的数据结构不合理，所以首先重构数据库数据结构。将学校和班级分离成两个不同的collection，在班级的collection中引用所属学校、学生和老师。

为了实现充分精简、通用、扩展与合理，我打算使用本文一开始提到的[express-restify-mongoose](http://florianholzapfel.github.io/express-restify-mongoose/)，哈哈，不要说我偷懒，之前不能用这个框架是因为它不支持嵌入式文档（MongoDB本身就不支持），现在将文档分离了，所以可以直接使用这个框架。
最具有吸引力的是这个框架支持collection之间的ref引用功能，我可以通过配置Schema来直接使用populate参数取出ref引用的对象，如：

>  /rooms/:roomId?populate=students,school
>  返回的班级对象包含完整的学校对象和学生对象

当然这个框架的也是有局限性的；比如中间件(middleware)是直接配置在Model上的，不能单独配置不同HTTP方法使用不同的中间件；再如这个框架中使用了大量的promise，使得node调用栈(Call stack)吃紧，一不小心就会将node的调用栈调满(maximum call stack size exceeded)。
基本能满足需要，具体如何看使用效果吧～～

需要注意的是，由于我们这个数据模型是关系模型，不同collection之间是存在联系的，比如删除一个学校的话，其下所有班级也应该全部删除，所以需要重写删除学校的API方法。

……此处省略10000字重构(xie)过程……

国庆假期的时候看了一篇文章《[HTTP API Design Guide](https://github.com/interagent/http-api-design)》，作者结合[Heroku Platform API](https://devcenter.heroku.com/articles/platform-api-reference)来详细说明设计HTTP API需要注意的问题，其中包含大量的HTTP header的运用和优化（又要去恶补了～～）以下是目录翻译（鄙人英语比较渣，欢迎斧正）以及我自己的一些看法：

* Foundations——基础
    * Require TLS——需要使用TLS(HTTPS)
    我的看法：  进行点对点加密可以很好的保障用户信息安全，不过这个和API设计关系不大。
    * Version with Accepts header——Accept头中包含版本信息
    我的看法：对API进行版本控制可以让升级变得更加灵活，在保留旧API的基础上现行开发和上线新API，依赖于此API的众多功能可以自行安排升级的开发日程，没必要等待API以及依赖这个API的所有功能都升级完毕之后同一上线，整体的开发和产品上线的效率。不过restify是通用的框架，没有真正的API版本概念，或许以后决定放弃restify之后这东西会大放光彩。
    * Support caching with Etags——支持Etag缓存
    我的看法：Etag是HTTP协议的一部分，专门用来进行缓存验证；以往的概念中，通常只有静态文件和页面需要缓存，如果可以借助Etag技术将HTTP API也进行缓存处理，合理运用的话可以明显降低服务器负载和提升用户体验，但其背后的算法也是相当复杂的，可以以后Hack一下。
    * Trace requests with Request-Ids——使用Request-Id来追踪请求
    我的看法：这种技术貌似只有Heroku在用，说是记录每次请求的ID有助于Debug程序，个人感觉没啥大用处，有待观望。。。
    * Paginate with ranges——使用range头来实现分页
    我的看法：分页功能在restify中是通过URL参数来实现的，指定当前一页的内容从第几条数据开始到第几条结束。使用HTTP协议中的Range头不太直观，而且由于Range是直接控制字节数的，所以实现起来也比较复杂。
* Requests——请求
    * Return appropriate status codes——返回适当的状态码
    我的看法：restify中虽然已经配置好了返回的状态码，但直接去W3C的网站上扒[Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)仔细看一看还是有好处的。
    * Provide full resources where available——提供完整的可用资源
    我的看法：对于我的系统来说即是提供尽可能完整的JSON，这样确实使HTTP API更加具有通用性，但是如果某些字段是安全敏感性的，那么可能就要费一费脑筋了，而且每次都提供完整信息对于比较特殊的数据结构来说可能会浪费内存和网络资源，需要权衡。
    * Accept serialized JSON in request bodies——接收请求体中的序列化的JSON
    我的看法：不多说，我的API一律使用JSON来传输数据。
    * Use consistent path formats——使用一致的路径格式
    我的看法：保持一致，容易管理。最好的实现方式是使用统一的API框架；当然不用框架的话一定要制定一个统一的API的定义规则，方便开发和管理。
    * Downcase paths and attributes——将路径和属性都转化成小写
    我的看法：不知道都用小写有何优势，文章中作者推荐使用下划线“_”链接单词，说这样的话在程序中可以直接用来做变量名，而且作为JSON的字段名(key)的时候也不必非要写双引号。其实只要有自己的统一的风格就好，没必要完全照搬，有些人就很习惯小驼峰。
    * Support non-id dereferencing for convenience——为了方便支持非ID的请求值
    我的看法：restify支持通过URL参数直接使用HTTP API进行数据查询，功能性很强。
    * Minimize path nesting——尽量减少路径嵌套
    我的看法：这恰好是我重构系统的目标之一，既然每一个数据对象都有唯一的分类和ID的话，完全不需要嵌套路径，只需要使用/分类/:ID就可以直接定位到这个对象了，有助与保持HTTP API的一致性和简捷。
* Responses——返回
    * Provide resource (UU)IDs——提供资源ID
    我的看法：其实作者想说的是全球唯一的ID（如UUID），这样有助于和其他（企业）的服务进行对接。但是究竟是需要每一个数据对象都要用UUID编号还是只是在对外结构使用UUID编号，具体还要看需求。
    * Provide standard timestamps——提供标准的时间戳
    我的看法：大多类型的业务逻辑都需要数据对象有创建时间和修改时间。
    * Use UTC times formatted in ISO8601——使用ISO8601标准中的UTC时间格式
    我的看法：千万不要用本地时间，另外也要确定服务器环境使用的是UTC时间，特别是国际化和与其他（企业）的系统协作的时候，使用UTC时间可以避免混乱和莫名其妙的BUG。
    * Nest foreign key relations——嵌套外键引用
    我的看法：怎么说呢？嵌套问题是我之前遇到的最大的坑，明确好嵌套关系非常重要，特别是MongoDB这样不支持嵌入式文档操作的情况。我的实现方式是使用Mongoose和restify自带的ref引用功能，可以使用一条请求同时拿到相关联的多个对象，返回的JSON表现为一个主要对象嵌套多个子对象。
    * Generate structured errors——生成结构化的错误信息
    我的看法：这个非常重要，我还没有使用，实际上这应该是一个团队乃至一个公司的通用标准，自然地也要有一份详尽的说明文档，提高开发和测试的效率。
    * Show rate limit status——显示速率限制状态
    我的看法：避免熊孩子的DOS攻击，限制每个客户端的请求频率，个人建议结合缓存机制，取得用户体验以及服务器性能的双赢。
    * Keep JSON minified in all responses——压缩返回的JSON
    我的看法：不光要压缩返回的JSON，最好把所有返回的资源都进行压缩，在HTTP自带的压缩协议(如gzip)的基础上，将HTML、Javascript、CSS文档都进行压缩，一方面大大节省带宽资源和提高访问速度，另一方面由于浏览器是直接解释页面代码的，代码的篇幅直接影响浏览器的解析速度。
* Artifacts——额外需要的工作
    * Provide machine-readable JSON schema——提供机器可读的JSON模式
    我的看法：restify是基于Mongoose的，Mongoose在定义数据库对象的时候是使用Schema的，结合不同的配置可以使复杂的数据持久化操作变得简单和清晰，同时也可用于数据合法性的限定，以及IDE的对象成员预声明。
    * Provide human-readable docs——提供人可读的文档
    我的看法：文档不应该使形式主义的，在开发过程中翻阅文档会大大提高开发效率，否则东问西问、重读代码会浪费整个团队的很多时间。
    * Provide executable examples——提供可运行的例子
    我的看法：团队内部的话，这点不是很必要，在开发框架和通用库的时候才真正必要，多少文字描述也比不过一个Demo。
    * Describe stability——描述稳定性
    我的看法：有必要作为文档的一部分，管理者可对整个项目或系统的状态一目了然，也便于让阅读者来了解到可能会存在哪些已知的问题。
