---
layout: post  
title:  "Angular2对抗React：一场流血的争斗"  
category: news  
author: "yinxin630"
---

原文：[点击打开][1]  
翻译：[碎碎酱][2]

Angular2已经发布了[Beta版本][3]，似乎将要成为2016年的热点框架，现在是时候与React一决高下了，让我们来看看它如何对抗2015年的宠儿：[React][4]。

声明：我很喜欢使用Angular1工作，但是在2015年切换到了React，我刚刚发表一篇对于React和Flux的[课程][5]（免费），所以，我是带有偏见的，但是我会同时批判它们。

![Angular大战React][6]

## 你是在比较苹果和猩猩

没错，是的，Angular是一个框架，而React仅仅是一个库，一些人说这种差异使得比较它俩是不合理的，完全不是！

选择Angular还是React就像你选择购买一台现成的电脑还是购买零件自己去组装。

这篇文章会从两个方面来考虑它们的优势，将React的语法和组件模型与Angular相比较，这就像对现成的计算机CPU和原始CPU作比较。

## Angular2的优势

让我们先来看看Angular2相对于React的优势。

### 较少的技术选型

由于Angular是一个框架，它提供了很多有长远意义的见解，并且暴露出了大量功能。而使用React的话，你通常会使用一些其它的现存库来共同构建一个真正的应用，你可能需要路由组件、单向数据流组件、API调用组件、测试组件、依赖管理组件以及其它组件。你会有十分多的选择，这就是为什么React会有如此多的starter kits的原因（我曾经写过两个：[react-flux-starter-kit][7]和[react-slingshot][8]）。

Angular暴露出了更多的功能，这可以帮助你快速的开始使用Angular，而不需要在各种技术选型中做抉择，这种强制的一致性也可以使得新员工更快的融入公司的工作中，使得开发者更容易转移到不同的团队中。

我很佩服Angular的核心团队拥抱TypeScript，这引出了下一个优势…

### TypeScript = 清晰的路径

确实，TypeScript并不被所有人喜爱，Angular2选择使用类JavaScript的混合语法开发是一个巨大的成功。网络上的React示例的不一致使人感到沮丧，它使用ES5和ES6的数量大致持平，并且它现在提供了[三种不同的方式][9]来定义组件，这使得后来者十分迷惑（Angular拥抱修改而不是继承 — 很多人认为这是正确的选择）。

在Angular2不需要依赖TypeScript之后，Angular的核心团队仍然继续拥抱它，并默认在文档中使用TypeScript，这意味着过去的示例和开源项目会使人感到熟悉和一致。Angular已经提供了[清楚的示例][10]来说明如何使用TypeScript编译器，（但不可否认，[并不是每个人都拥抱TypeScript][11]，但是我怀疑它不久就会成为事实上的标准）。这种一致性有助于避免混淆，并且减少了React入门的那些额外负担。

### 减少客户流失

2015年是JavaScript疲惫的一年，React是一个关键的贡献者，而且React还没有发布1.0版本，将会可能会有重大改变。React的生态系统仍在快速的发展，特别是长长的[Flux][12]和[路由组件][13]的列表，这意味着，任何你现在写下的代码都可能在1.0版本发布后必须经历重大改动。

对比来看，Angular2的改动有条不紊，是一个成熟的、全面的框架，因此它在发布后不会有大量的痛苦的改动，而且作为一个完整的框架，在你选择Angular之时，你可以在未来一直信任你的团队。而React，你需要去比较一堆不同的、快速迭代的、开源的库，来综合考虑哪些可以很好的一起工作，这是一个费时费力、令人沮丧，并且永无止境的工作。

### 更多的工具支持

如你所见，我认为React的JSX是一个巨大的成功，你需要选择支持JSX的工具，React早已十分流行，工具支持已经不再是一个问题，但是新工具譬如IDE和linter在一开始都不太可能支持JSX。Angular2的模板标记储存在字符串或者单独的HTML文件中，所以它不需要特殊的工具支持（Angular的字符串模板在过程中已经被智能的解析），也就是说，Angular有着它自己的解析字符集的方法。

## React的优势

好的，再让我们来看看React这部分。

### JSX

JSX是一种类似HTML的语法，最终编译为JavaScript，标签和代码混合在一个相同的文件里，这给了你一种组织组件的函数和变量的方法。相反，Angular的基于字符串的模板有着一些常见的缺陷：在任何编辑器里都没有代码高亮、自动补全和运行时检查，你通常会期待较少的错误消息，但是，Angular团队创建了他们自己的HTML解析器来修复这个问题（鼓掌）。

如果你不喜欢Angular的基于字符串的模板，你可以把模板移动到单独的文件，但是你会遭遇被我叫做"the old days"的情况：在你的脑海中同时思考两个文件，并且没有代码补全和运行时代码检查推断。如果你使用React的话，这些并没有什么大不了的。组件单独放在一个文件中，具备编译时检查，这是JSX如此特别的理由之一。

![Contrasting how Angular 2 and React handle a missing closing tag][14]

获取更多关于JSX为何如此成功的信息，请查看[JSX：硬币的另一面][15]。

### 快速明确的错误提示

当你在React的JSX中制造一个错误时，它不会被编译，这样是美好的，这意味着你立刻就能知道哪一行有错误，如果你忘记了关闭标签或者引用了错误的属性，它会立刻提示你。事实上，JSX编译器在发生错误时指定行数，这使得我们可以快速开发。

相反，你在Angular2中错误的输入一个变量，什么都不会发生，Angular2会默默的运行下去，这使得开发速度被放缓，我加载了app并考虑为什么我得数据没有被显示出来，这不是一件开心的事。

### React以JavaScript为中心

这是React和Angular根本上的不同，不幸的是，Angular2中仍残留着以HTML为中心而不是以JavaScript为中心。

*Angular 2 continues to put “JS” into HTML. React puts “HTML” into JS.*

我不能更多的强调这种分歧带来的影响，它从根本上影响着开发体验。Angular的以HTML为核心设计仍是其最大弱点，正如我所说，[JSX：硬币的另一面][16]，JavaScript远远比HTML强大，因此，更多的JavaScript支持比更多的HTML支持要更符合逻辑，HTML喝JavaScript需要以某种方式绑定在一起，React的以JavaScript为核心的目标在根本上比Angular、Ember和Knockout的以HTML为核心更优秀。
这就是原因。

### React的JavaScript核心观使得设计 = 简单

Angular2延续了Angular1的使HTML更强大的目标，所以你不得不利用Angular2那独一无二的语法来完成一些累死循环和判断的简单任务。比如，Angular2同时提供了两种数据绑定语法，而且不幸的是，它们完全不同：

```
{{myVar}} //One-way binding
ngModel="myVar" //Two-way binding
```

在React中，绑定标记不会被改变（它在任意位置按照我所理解的那样被处理），在任意一处，它都是这样的：

```
{myVar}
```

Angular支持这样的内置模板：

```
<ul>
  <li *ngFor="#hero of heroes">
    {{hero.name}}
  </li>
</ul>
```

上面的代码段遍历heroes数组，我有很多个关注点：通过星号来标识定义了一个master template”，这样含义是模糊的，在hero前的星号定义了一个本地模板变量，这个核心概念产生了不必要的全局污染（你可以首选使用var）。ngFor通过Angular特殊的属性给HTML添加了循环语法，将Angular2的语法与React的纯JS语法相比：公认的React做的更好。

```
<ul>
  { heroes.map(hero =>
    <li key={hero.id}>{hero.name}</li>
  )}
</ul>
```

因为JS天生支持循环，React的JSX可以很容易的利用强大的JS功能，比如map、filter。

只要你读过[Angular2小计][17]，你就会知道，那不是HTML，那也不是JavaScript，它就是Angular。

*To read Angular: Learn a long list of Angular-specific syntax.
To read React: Learn JavaScript.*

React的独特之处在于它的语法和简单的概念，再来对比下当今流行的框架/库：

```
Ember: {{# each}}
Angular 1: ng-repeat
Angular 2: ngFor
Knockout: data-bind=”foreach”
React: JUST USE JS. :)
```

除了React的其它框架的特性都是在替代一些原有的或者在JavaScript中不重要的事情：比如循环。React的魅力之处在于，它拥抱强大的JavaScript来处理标记，所以不需要各种奇奇怪怪的新语法。Angular2的古怪语法还包括单击事件绑定：

```
(click)=”onSelect(hero)"
```

相反，React简单的使用JavaScript语法：

```
onClick={this.onSelect.bind(this, hero)}
```

并且，自从React包含一个综合的事件系统后（就像Angular2的做法），你不必担心这样的行内处理事件的效率问题。

### 顶级开发体验

JSX代码的自动补全支持、编译时检查和完善的错误消息，创建了一个卓越的开发体验，同时节省了你的时间和代码录入工作。再结合热加载、时间旅行，你就具备了其它框架不能达到的极速开发体验。

### 文件大小

下面是常见的知名框架/库的大小：

```
Ember: 580k
Angular 2: 565k (759k with RxJS)
React + Redux: 204k
Angular 1: 145k
```

为了做一个真实的比较，我同时在Angular2和React中构建了Tour of Heroes应用，结果呢？

```
Angular 2: 764k minified
React + Redux: 216k minified
```

简单的比较，Angular2有着React+Redux三倍的大小。

我承认过于鼓吹文件大小是没意义的：

```
Large apps tend to have a minimum of several hundred kilobytes of code — often more — whether they’re built with a framework or not. Developers need abstractions to build complex software, and whether they come from a framework or are hand-written, they negatively impact the performance of apps.
Even if you were to eliminate frameworks entirely, many apps would still have hundreds of kilobytes of JavaScript. — Tom Dale in JavaScript Frameworks and Mobile Performance
```

Tom是对的，像Angular、Ember这样的框架如此之大是因为它们提供了太多的功能。

然而我更担心这个：很多应用并不需要框架所提供的所有内容，当今世界，越来越趋向于拥抱微服务、微应用和[单职责模块][18]，React谨慎的给你提供了一些必备功能，在NPM社区有着超过[20万个][19]模块，它们给你提供了更强大的功能。

### React拥抱Unix哲学

React是一个库，它清楚的站在了类似Angular和Ember这样大型的、综合的框架的对立面，因此，当你选择了React之时，你就可以自由的选择最适合解决你的问题的库。JavaScript发展的很快，并且React允许你替换应用中的一部分使用更好的选择来代替，而不必等待你所用的框架来解决这个问题。

Unix已经禁住了时间的考验，因为：

```
The philosophy of small, composable, single-purpose tools never goes out of style.
```

React是一个专注的、单一目标的有用工具，已经被[很多大型网站][20]采用。

  [1]: https://medium.com/@housecor/angular-2-versus-react-there-will-be-blood-66595faafd51#.bskqff1mp
  [2]: http://www.suisuijiang.com
  [3]: https://angular.io/
  [4]: https://facebook.github.io/react/
  [5]: https://www.pluralsight.com/courses/react-flux-building-applications
  [6]: https://cdn-images-1.medium.com/max/800/1*MRPl_SNuRGJchb6eOAnkSA.jpeg
  [7]: https://github.com/coryhouse/react-flux-starter-kit
  [8]: https://github.com/coryhouse/react-slingshot
  [9]: http://jamesknelson.com/should-i-use-react-createclass-es6-classes-or-stateless-functional-components/
  [10]: https://angular.io/docs/ts/latest/quickstart.html
  [11]: http://angularjs.blogspot.com/2015/09/angular-2-survey-results.html
  [12]: https://github.com/kriasoft/react-starter-kit/issues/22
  [13]: https://github.com/rackt/react-router
  [14]: https://cdn-images-1.medium.com/max/1600/1*ivDnyMP63YJBBGKCNyRUzQ.png
  [15]: https://medium.com/@housecor/react-s-jsx-the-other-side-of-the-coin-2ace7ab62b98#.5007n49wq
  [16]: https://medium.com/@housecor/react-s-jsx-the-other-side-of-the-coin-2ace7ab62b98#.5007n49wq
  [17]: https://angular.io/docs/ts/latest/guide/cheatsheet.html
  [18]: http://www.npmjs.com/
  [19]: http://www.modulecounts.com/
  [20]: https://github.com/facebook/react/wiki/Sites-Using-React