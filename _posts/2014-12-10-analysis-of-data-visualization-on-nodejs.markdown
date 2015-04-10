---
layout: post
title: "基于 Nodejs 的数据分析"
category: tech
author: song940
---

我们有很多以行为单位类似日志的数据，我们想要从中分析出有价值的数据展现给我们的用户。

```javascript
{ event: 'FinishLesson', UserId: '544642cc36704cf31acbf123', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'Login', 		UserId: '544642c2e02c67ed1a10e1f7', timestamp: ISODate("2014-10-21T19:26:04.064Z") }
{ event: 'EnterHome', UserId: '54059540ef73b666603a6916', timestamp: ISODate("2014-10-21T19:27:15.449Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '54059408ef73b666603a6908', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '54059540ef73b666603a6916', timestamp: ISODate("2014-10-21T19:27:33.503Z") }
{ event: 'FinishLesson', UserId: '5446428b36704cf31acbeea5', ChapterId: '54059540ef73b666603a6916', LessonId: '53f56f2f6dc0068a6f107f98', timestamp: ISODate("2014-10-21T11:27:15.449Z") }
```

但是这有几个问题：

1. 这些数据可能不正确
2. 这些数据可能不完整

下面来详细聊聊以上几种情况具体是怎么回事。

## 关于数据不正确的故事

由于用户客户端浏览器的不稳定性可能导致数据记录不完整，这有可能是因为网络问题，也有可能是因为我们的逻辑问题。反正最终我们收集到的数据是这个样子的。

```javascript
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
```

显然这个事件没有了 `ProblemId` 这个字段，千万不要以为这只是个例，当数据足够多的情况下这样的数据就会越来越明显。

再举个例子：

```javascript
{ event: 'StartProblemSet', UserId: '538fe05c76cb8a0068b14031', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '8a5bcfcbef851125122dc4a8e9887469', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '8a5bcfcbef851125122dc4a8e9887469', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '8a5bcfcbef851125122dc4a8e9887469', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'StartProblemSet', UserId: '538fe05c76cb8a0068b14031', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '8a5bcfcbef851125122dc4a8e9887469', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '8a5bcfcbef851125122dc4a8e9887469', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'AnswerProblem', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', ProblemId: '8a5bcfcbef851125122dc4a8e9887469', timestamp: ISODate("2014-08-25T22:14:45.976Z") }
{ event: 'FinishProblemSet', UserId: '538fe05c76cb8a0068b14031',ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', CorrectCount: 5, timestamp: ISODate("2014-08-25T22:14:45.976Z") }
```
细心的你一定发现了这个事件流过程中少了一个 `FinishProblemSet`，这种情况和上面提到的数据不正确的情况很像，但是他却影响了这个过程中的所有数据。

那么这种错误的数据显然会给我们后面的计算过程带来干扰。

## 关于数据不完整的故事

很多情况下这种 Track 记录的数据不可能面面具到。一般也就记录一些 ID， 但是计算过程中是需要一些完整的数据信息才可以的。

比如下面的这个例子：

```javascript
{ event: 'FinishProblemSet', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', CorrectCount: 5, timestamp: ISODate("2014-08-25T22:14:45.976Z") }
```

只记录了一个 `CorrectCount` ，但是我们要计算他的正确率怎么办呢？

所以就要通过他的 `ChapterId` 和 `LessonId` 来得到他下面的所有 `Problems` 的数量。然后填充到这条数据中。得到下面的结果：

```javascript
{ event: 'FinishProblemSet', UserId: '538fe05c76cb8a0068b14031', ChapterId: '538fe36a76cb8a0068b14036', LessonId: '538fe2a576cb8a0068b14035', CorrectCount: 5, ProblemSize: 7 , timestamp: ISODate("2014-08-25T22:14:45.976Z") }
```

## 初步实现

我们需要计算 `Room` 和 `Student` 两个维度的数据情况。

通过分析我们不难发现，`Room` 的数据显然是依赖于 `Student` 的计算结果的。

那么我们打算先计算 `Student` 的数据，然后再计算 `Room` 的数据。

于是我们构造了这样的一条请求

```javascript
GET /tracks?$and=[{"data.event":"$event_key"},{"data.properties.distinct_id":"$username"},{"$or":[{"data.properties.usergroup":"student"},{"data.properties.roles":"student"}]}]';
```

这会返回一组 Tracks 数据，我们需要对这些数据进行筛选和过滤以及数据的补全等操作。

通过请求不同的事件我们可以计算出这个 `Student` 的各种方面的学习情况。

当所有学生的情况都已经计算完成，再将这些学生按照班级分组。以组的形式计算班级的整体学习情况。


## 关于缓存

由于计算过程中经常需要：

+ 课程数据用来填充 Track 的信息
+ 获取某个班级中的所有学生
+ 在 `UserName` 和 `UserId` 之间互相转换

所以为了避免频繁的向数据库查询请求，我们将这些常用数据缓存起来以提高响应速度。

## 关于增量计算

运行一段时间发现虽然传统的工作模式可以很好的实现数据计算任务，但是最大的问题就是每次计算的时间实在太长了。

所以我们的模式就必须要改变，那么如何解决这个问题呢？让我们来分析下原因。

计算时间长的主要原因是：每次我们都需要做同样的事情（补全信息，剔除错误数据 。。。）

那么有没有可能让这些事情只做一次，避免做重复性的工作呢。

还有一个问题是即使我们只想计算某一特定的事件类型，也要从头到尾过一遍才能收集到所有需要的信息。

那么有没有办法解决呢


办法总是有的，我们通过将各数据的信息补全剔除，然后进行分类整理等一系列过程，最终存储到一个中间单元中。

然后每次有新的 `Track` 信息进入以后就走一次上面的流程。这样数据量就非常小。也就是说每次只计算个位数的 `Track` 数据， 而不是以前的 `3,000,000` 条数据同时计算。

但是上面提到数据存储到了一个中间单元中了，如何将其转化成最终的数据呢？

经过测试我们发现，如果数据是完整的并且拥有我们想知道的一些信息，那么计算过程就非常轻松。只需要按照不同的维度做一些简单的运算就可以了。


## 总结

面对大量数据的计算时，我们需要考虑的问题很多，因为什么样的数据都有。

由于我们的数据是通过向 `API` 发送请求后获得的 `JSON` 格式数据，那么我们回来要做 `JSON.parse` 反序列化，但是当数据量过大时 `JSON.parse` 是抗不住的，所以还要小心的控制每次请求的数量级。

还有就是调试起来很麻烦，面对如此多的数据，你很难准确的找出到底是哪里出了问题。
