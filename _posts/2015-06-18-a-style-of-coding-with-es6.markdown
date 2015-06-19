---
layout: post
title:  "a style of coding with es6"
category: tech
author: "link"
---

1 - promise

```javascript
when.promise(function(resolve, reject) {
        Chapter.create(req.body.chapter, function(err, chapter) {
            (err || !chapter) ? reject({ error: err }) : resolve(chapter);
        });
    }).then(function(chapter) {
        //if need to bind
        if(!req.courseVersion) return chapter;
        return when.promise(function(resolve, reject) {
            req.courseVersion.save(function(err, saved) {
                (err || !saved) ? reject(err) : resolve(chapter);
            });
        });
    }).then(function(chapter) {
        res.status(200).json(chapter);
    }).catch(function(err) {
        res.status(500).send(err);
    });
```
对数据库你可以不使用回调，直接返回promise, 所以你可以写成这样，消掉所有回调嵌套，和promise 嵌套

```javascript
when.promise(function(resolve, reject) {
        return Chapter.create(req.body.chapter);
    }).then(function(chapter) {
        //if need to bind
        if(!req.courseVersion) return chapter;

        return req.courseVersion.save()
    }).then(function(chapter) {
        res.status(200).json(chapter);
    }).catch(function(err) {
        res.status(500).send(err);
    });
```

 2 - generator

但还可以优化吗？可以让我们的访问数据库结果在一个上下文空间吗？让我们用se6的generator进行尝试

```javascript
co(function *() {
    var chapter = yield Chapter.create(req.body.chapter)
    if(!req.courseVersion) return chapter;

    chapter = yield req.courseVersion.save()
    res.status(200).json(chapter)
}).catch(function(err) {
    res.status(500).send(err);
});
```
yes, got it!

那么可以相同的方实异步调用吗？ try it

一下是部分数据平台的留存借口代码

```javascript
co(function *() {
    result.firstDay = self.count({
        user: {$in: users},
        createdBy: date,
        events: {
            $in: [Event.ejectFinishElementary,
            Event.ejectFinishAdvanced,
            Event.finishChallenge]
        }
    });

    result.nextDay = self.count({
        user: {$in: users},
        createdBy: moment(date).add(1, 'days').format('YYYY-MM-DD')
    })

    result.in7days = self.distinct('user', { // 7 days range
        user: {$in: users},
        createdBy: {
            $gte: moment(date).add(1, 'days').format('YYYY-MM-DD'),
            $lte: moment(date).add(7, 'days').format('YYYY-MM-DD')
        }
    })

    return yield result
})
```
see it ? result 包含了多个调用数据接口函数，yield 可以保证同步的同时并发执行多个函数，并且以object类型返回结果

那么，并发操作还可以嵌吗？　try it again? 

```javascript
var generator = co.wrap(function *(id, users) {
    result.firstDay = self.count({
        user: {$in: users},
        createdBy: date,
        events: {
            $in: [Event.ejectFinishElementary,
            Event.ejectFinishAdvanced,
            Event.finishChallenge]
        }
    });

    result.nextDay = self.count({
        user: {$in: users},
        createdBy: moment(date).add(1, 'days').format('YYYY-MM-DD')
    })

    return yield result
})(id, users)

var generator1 = co.wrap(function *(id, users) {
    var results = yield self.distinct('user', { // 7 days range
        user: {$in: users},
        createdBy: {
            $gte: moment(date).add(1, 'days').format('YYYY-MM-DD'),
            $lte: moment(date).add(7, 'days').format('YYYY-MM-DD')
        }
    })

    return results
})(id, users)

var generators = [generator(item._id, item.users), generator1(item._id, item.users)]
var results = yield generators;

```
（　⊙ｏ⊙　）哇,yield　还可以返回generator的结果
，，所以，从这里可以看出，理论上可以用yield写出任何层次的同步＋异步代码的组合，而且还事扁平化

3. map reduce filter join

数据库的聚合操作利用了　map reduce 模型，同样我们也可以重复利用这类模型，就如同递归的世界，重复自身
代码节选如下：
```javascript
userMapList.reduce((a, b) => {
    b.users = b.users.filter(item => a[item] === undefined)
    b.users.forEach(item => a[item] = true)
    return a
}, {})

userMapList = userMapList.reduce((a, b) => {
    a[b._id] = b.users.length
    return a
}, {})
```
这种是重复聚合操作，类似mongo的连续２个group管道操作

```javascript
var users = results.filter( item => !(item.count == 1 && item.registDate >= startDate) ) // exclude activated users
var userMap = users.reduce((a, b) => {
    ++a[ (b.count >= 12) ? 'd12+' : 'd'+ b.count ]
    a.allCount += b.count
    return a
}, {d1:0,d2:0,d3:0,d4:0,d5:0,d6:0,d7:0,d8:0,d9:0,d10:0,d11:0,'d12+':0,allCount:0})
```

结合过滤管道进行reduce 操作
```javascript
userMapList.reduce((a, b) => {
    b.users = b.users.filter(item => a[item] === undefined)
    b.users.forEach(item => a[item] = true)
    return a
}, {})
.map((item, n) => {
    var userMap = userMapList[n];

    for (var key in item) {
        var value = item[key]
    }
    item.date  = userMap._id
})
```
标准map reduce 操作
等等，所以使用Array给我们提供的 filter, map, reduce模型， 基本就可以处理绝大部分数据操作

nosql 是无关系模型，该如何处理结果之间的关联呢？

数据平台代码节选：
```javascript
var schools = _.hashFullOuterJoin(school.havclass, accessor, school.noclass, accessor)
    .map(item => {
        item.allCount = (item.havclassCount || 0) + (item.noclassCount || 0)
        return item}),
    schools1 = yield School.find({_id: {$in: schools.map(item => item._id)}})
    .select('name').lean().exec();

schools = _.hashLeftOuterJoin(schools, accessor, schools1, accessor)
.sort((a, b) => b.allCount - a.allCount)
.map((item, n) => {
    item.noclassCount = item.noclassCount || 0;
    item.havclassCount = item.havclassCount || 0;
    item.order = ++n
    delete item._id;
    return item}).slice(0, req.body.limit)

res.json({result: schools})
```

可以看到　hashFullOuterJoin，　hashLeftOuterJoin　就是关系数据库的多表操作，这里模拟了这种关系型数据库各种join操作，
以达到组合数据的目的，　这里想到　postgresql正式结合了两者的优点

4. debug

关于调试，从generator,我们所有的异常都会被汇到　catch处理函数，不需要在各处回调判断，而且，还可以抛出自定义异常，
或者使用标准　try catch　捕获异常，防止程序意外崩溃，让程序专心处理业务逻辑

5.parallel universe
到这里，已经联想到李狗蛋的平行宇宙理论，所有时间在同一水平上并发执行，通过虫洞(yield) 来互相通信，oh yeah!
当然，代码中充斥着　arrow function等额外的　es6特性，大家就自行阅读了，关于arrow function，核心还是　Lambda　表达式，
一定要了解下　Lambda演算，对程式设计有很多启发，　《the little scheme》是个不错的选择，丁丁的ｓｉｃｐ 也行

6. tip
just do one thing per function
As little as possible nesting level
Process oriented programming fist, oop second, don't oo too earlier, y'll not see the whole scence.
like write scripts, refactor step by step...
UML - thinking of data flow -> clear data model(data structure) & whole image in mind (第五项修炼, 系统思考)
naming　style - very importaint
less third party module, unless y're familier its source code
must add semicolon before (, [, +, -, / if y decide not use semicolon....

7. if we responsible of ourself code -> less code, more Elegant, get more free time to do y love

8. 看清问题的根源和因果关系，最终消灭问题

9. small is beautiful

美，能做的事越多，代码量却越少，他必须完美把握住事物的本质，否则就会有许多无法修补的特例。在修改代码的时候，你必须用
心灵之眼看见代码背后表达的事物，他们不怎么用调试器，只是用眼睛看着代码，然后闭上眼，脑海里浮现出其中信的流动，所以他们经常一动手就能改到
正确的地方　。。。
