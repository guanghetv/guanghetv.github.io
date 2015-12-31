---
layout: post
title:  "React 懒人教程"
category: tech
author: "yinxin630"
---

原文：[查看原文][1]  
翻译：[碎碎酱][2]

我曾经努力了很长一段时间来尝试理解什么是React，以及React适合用于软件开发的哪一部分，这篇文章就是关于我曾经想要获取的那些信息。

##什么是React？

React和Angular, Ember, Backbone或者其它框架比较起来怎么样？我该怎么样处理数据？我该怎么样与后端联系起来？什么是JSX？什么是组件？

停！

先不要考虑这些问题！

##React仅仅是View层

React经常与其它JavaScript框架一起被提及，但是像『React VS Angular』这样的比较是没有意义的，因为它们并不是可以直接比较的事物。Angular是一个完整的框架（包含View层），React却不是，这就是React使人晕头转向、难以理解的原因，它只是一个新兴的、一个完整框架生态系统中的一环，它仅仅是View层。

React给了你提供了一种模板语言和一些钩子函数来渲染真实的HTML，它仅包含唯一输出：HTML。你会封装HTML/JavaScript代码并称之为组件，组件支持在内存中存储它自己的状态（比如在Tab View中哪个标签被选中），但是最终仅仅被输出为HTML。

你绝对不可能仅仅使用React来构建一个功能完备的动态WEB应用，我们将在接下来学习更多的内容关于它为什么不可以。

##优点

在用React工作一段时间后，我发现了三个很重要的、有价值的点。

###1.你总是可以仅仅查看一个源文件就得知组件会如何被渲染

这可能是最重要的好处，甚至尽管它与Angular的模板没有什么不同，让我们使用一个真实的例子来说明

假如你需要在用户登录后修改网站头部的用户名显示，如果你没有使用JavaScript MVC框架的话，你可能会这样做：

```
<header>  
    <div class="name">
        Not Logged In
    </div>
</header>  
```

```
$.post('/login', credentials, function( user ) {
    // Modify the DOM here
    $('header .name').text( user.name );
});
```

我可以用我以往的经验来告诉你，这个代码会摧毁你的生活和你同事的生命，你如何去调试输出？谁来修改头部内容？还有谁可以访问头部的HTML代码？谁控制着修改用户名显示的代码段？这种DOM操作就像goto关键字一样，使你很难去推算代码逻辑。

下面这段代码是如何使用React来做这件事：

```
render: function() {  
    return (
        <header>
        { 
            this.state.name ?
            this.state.name :
            <span>Not Logged In</span> 
        }
        </header>
    );
```

我们可以很直接的看出这个组件将如何被渲染，只要你知道组件的状态，你就会知道组件渲染后的输出，你不需要追溯程序流，当工作于复杂的应用程序时，特别是处于团队中时，这一点十分重要。

###2.将HTML/JavaScript代码封装在JSX中使得组件很容易被理解

上面这种怪异的将HTML/JavaScript混合编写的行为可能会使你畏惧，在过去很早的时候，我们已经适应了将JavaScript与HTML分离（譬如onClick处理过程），但是，现在你需要相信我，使用JSX来编写React组件是十分美好的。

传统上，我们将视图（HTML）与功能（JavaScript）分离，这导致我们会为一个网页编写一个整块的JavaScript文件包含页面所需的所有功能，这使你不得不去追溯复杂的工作流 JS > HTML > JS > bad-news-sad-time。

尝试将函数封装在便于使用的、独立的组件中会使你感到美好，并且会降低全局污染，你的JavaScript代码清楚的知道如何控制你的HTML代码，因此将它们揉合在一起是有意义的。

###3.你可以在服务端渲染React

如果你正在构建一个开放的网站或者移动端app，并且你是按照将其全部加载到客户端（render-it-all-on-the-client）的办法，那么你过去都做错了，仅在客户端渲染就是导致[SoundCloud][3]感觉很缓慢而[Stack Overflow][4]（纯粹的服务端渲染）却感觉很快的原因，你可以在服务端渲染React，而且你很应该这样做。

Angular和其它框架鼓励你使用PhantomJS来处理一些烦人的事情譬如页面渲染，基于用户代理为网络搜索爬虫服务，或者获取花钱来购买这种服务。

##缺点

不要忘记React仅仅是View层

1.它不包含下列内容

*  AJAX功能
*  数据表格
*  Promises
*  任何应用框架
*  任何实现上述内容的想法

在实际业务中仅仅使用React是无用的，更糟糕的是，就像我们将要看到的那样，这将导致每个人不断的重复造轮子

2.文档不友好，难以阅读，就像写给懒人的博客文章，请在[React Doc][5]中查看导航栏的第一部分。这里有三个分离的，相互矛盾的快速开始引导，我完全看晕了，它下面的侧边栏简直是我的噩梦，一条条很明显的内容不该出现在这里，比如"More About Refs" 和 "PureRenderMixin"。

##说说Flux

大概大多数的React开发者最恼人的部分就是Flux，它远远比React更容易使人晕头转向，Flux这个名字就是一个理解的障碍。

并没有Flux这个事物，Flux是一种概念，而不是一个库，这里是[Flux的实现][6]

Flux更像是一种模式，而不是一个框架

React并没有彻底改造过去40年来的UI构建知识，它是带着一些数据管理的新观念出现的。

Flux的内容简单讲就是，你的视图触发一个动作（在用户键入用户名之后），这个动作将会更新模型，然后模型触发一个事件，视图响应模型的事件，并使用最终的数据重新渲染自身，这就是Flux。

这个单向的工作流、观察者模式的设计可以保证你的源代码事实上的与存储/模型相分离，这样做是极好的。

Flux不好的一点是很多人都有自己的实现，没有一致的事件库、模型层、AJAX层以及其它内容，目前有着很多不同的Flux实现，并且它们彼此竞争。

##我该使用React吗？

简单讲：应该

###你该使用React的理由：

*  适合团队开发，健壮的、易于使用的UI和工作流
*  UI代码是易读的和便于维护的
*  组件化UI是WEB开放的将来，你应该现在就开始这么做

###在决定使用之前你需要思考更多：

*  React会非常明显的降低首页渲染速度，理解什么是state、什么是props和组件交互等内容并不简单，并且文档的内容使人迷惑，它将会是一个冲击，理论上，会大大加速你整个团队的效率。
*  React并不支持IE8或者更低版本，并且永远不会。
*  如果你的网站/应用并不需要太多的动态页面更新，你将会很小的益处而额外编写大量的代码。
*  你将会创造大量轮子，React还很年轻，并且目前还没有权威的方法来解决事件/组件交流。

  [1]: http://blog.andrewray.me/reactjs-for-stupid-people/
  [2]: http://www.suisuijiang.com
  [3]: https://soundcloud.com/
  [4]: http://stackoverflow.com/
  [5]: http://facebook.github.io/react/docs/getting-started.html
  [6]: https://github.com/facebook/flux