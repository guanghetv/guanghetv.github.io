---
layout: post  
title:  "React 荒野求生指南"  
category: Tech  
author: "lvBingo"
---

作者：吕骚娆

大家好，吕骚娆就是我，我就是美不胜收、智商爆表、无解的智商还有无解的颜艺，美貌与智慧并重，知识和美德的结合体的吕骚娆。我将在这篇文章中介绍我在使用React和Redux写code时的心得，嘻嘻嘻。

# 说明

流行的名词都是噱头，我们的目标是`没有蛀牙`

在这篇文章将介绍以下有关React的东西：

1. React的概念
2. 如何应用好React？
3. Redux的概念
4. Redux和React在一起时的组成部分
5. immutable-js和Redux

以上内容想到哪儿写哪儿。这篇文章不应该面向没用写过React和Redux的人，我假设你有一定的React编程基础，用React和Redux写过一些案例。我将不会介绍细节，但保证干货。不会出现屁用都没有的老生常谈的东西，但代价是你需要提前体验过React，这些内容才会加深你对React的理解。


# React？
`React本质上是一个view层`，但这个说法太过混淆和不负责任。React是一个状态机，通过状态的改变来调用Render事件重新计算和渲染Dom。

除此之外，使用React本质上是在做面向过程式编程，先按下不表。

状态鸡接受两种状态：`state`和`props`。`state`是一个React Component内部的状态；与之相对的`props`是Component接受的外部对其Component的状态。不论是`state`和`props`哪一个发生改变，都会调用这个Component的Render方法进行重新计算。

Render方法会渲染你想要的Dom，React围绕着Render方法进行创建了一套生命周期。这套生命周期共7个事件、两个初始化方法，按照一个React Component被执行的时间顺序为：

```
getInitialState  // 初始化 state
getDefaultProps  // 设置默认的 props，如果外部没用传递props则使用默认的props
componentWillMount  // 组件即将 Render 之前，不经常用到
render
componentDidMount   // 组件第一次 Render 之后
componentWillReceiveProps(nextProps)        // 组件接收到新的 props，要被 update 时。这个方法最大的作用就是你需要在 props 被更新的同时更新 state，调用 this.setState 方法不会触发两次 Render 事件
shouldComponentUpdate(nextProps, nextState)         // 你拿到即将被接收的两种状态，与现在的进行对比，你可以返回 false 来阻止这次组件被 update。当你 App 逻辑复杂，使用 Redux 传递对象时，你会经常用到这个方法。
componentWillUpdate(nextProps, nextState)   // 组件即将被更新，不经常用到，不应该在此处更新内部状态 state
render
componentDidUpdate(nextProps, nextState) // 当组件被更新之后，你可以在此处触发动画
componentWillUnmount    // 组件将被移除时，你可以在此处销毁setTimeInterval变量等。
```

React作为一个view层的精妙之处就在与完善的生命周期和组件和组件之间的协作方式，就是刚刚提到的面向过程式编程，接下来为你介绍组件之间协作时要`必须`遵守的哲学。

#如何应用好React？
你在创建一个React应用时，请务必谨记——将应用的组件分为`容器视图`和`表现视图`，在设计目录结构时，你可以这样设计：

```
react-app/src/ +  
        
        —— | app/                   放置 React 容器视图
            —— | exercise.js

        —— | components/            放置 React 表现视图
            —— | exercise/
                —— | dataLoading.js
                —— | header.js
                —— | mainContent.js
                —— | softkeyboard.js
                
        —— index.html               单页应用的基础html文件

        —— module.js                程序入口，初始化 React Router，调用容器视图

```

明白我的意思了吗？React组件是要和数据相连接的，数据的流向应该时从容器视图以传递`props`的方式流入表现视图的，举个例子，下面的图片是我的容器视图，它使用了Redux数据流。

<!-- 博客中实际相对路径 -->
![](/assets/images/1231672387142375462345245.png)

图片中如果有你看不懂的东西不要在意，我只是希望你知道在你的App规模变得复杂时，在`容器视图`和`表现视图`之间使用`props`的方式传递参数有多么必要。当你的数据流使用Redux进行管理时，它会让你的App井然有序，`容器视图`就想一个列表一样，将所有有关数据（这里指的数据还包括UI的状态）的操作罗列出来。但还不止如此，接下来就介绍Redux。


#Redux的概念
Redux是一种数据的管理方式的库，通过dispatch一个Action，Redux Reducer会处理这个Action，并将结果更新至store（在Redux中叫做state）。
使用Redux的好处在于，你会用Redux管理数据、React Component 之间跨组件的UI状态，你可以非常方便的添加[middleware](http://redux.js.org/docs/api/applyMiddleware.html)在每一个action被发起和数据更改后的状态，像这样：

<!-- 博客中实际相对路径 -->
![](/assets/images/1231378236428345245.png)

你还可以在发起Action后捕获出现的异常，将它们发送至服务器记录。你做test case时也比一般应用要简单的多，因为数据的更改全部通过Action，没有其它的更改办法。

#Redux和React在一起时的组成部分

在你使用React的`容器视图`与Redux store [connect](https://github.com/rackt/react-redux)之后，store一旦被更新，就会将新的数据以props的方式传递给你的`容器视图`，这个数据不一定是整个Store，而可以是一个预加工过的store，通过[Selector](https://github.com/rackt/reselect)进行计算后的数据。这是因为Redux提倡整个App只有一个Store的存在，但你可能有多个不相关的`容器视图`，你需要[Selector](https://github.com/rackt/reselect)来帮你预计算和缓存。

你的`容器视图`收到`props`后一旦被Render，你的`表现视图`也会接收到新的`props`，整个App被更新。

相对的，如果你不使用过程式编程的思想在`容器视图`和`表现视图`之间传递`props`，而是将store与每一个`表现视图`之间connect，将会极大造成效率问题，不适合构建大型应用，逻辑也不够清晰。

#immutable-js和Redux

Redux提倡在Reducer中计算时不要更改一份旧的数据，而是创建一份新的数据，也不要在Reducer外修改Store中的数据。深拷贝无疑将造成效率问题。所以我们的目光投向了immutable-js。

再也没有比immutable-js更适合和Redux进行一起服用的解决方案了。

immutable-js的[实现](https://en.wikipedia.org/wiki/Persistent_data_structure)类似于链表，添加一个新结点把旧结点的父子关系转移到新结点上，性能提升很多。由于是不可变的，可以放心的对对象进行任意操作。在React开发中，频繁操作state对象或是store，配合immutable-js快、安全、方便。


#结语
总的来说这篇文章并不是教你如何写出一个Demo，而是分享我在开发时的经验，希望这些经验在你开发时带给你灵感。