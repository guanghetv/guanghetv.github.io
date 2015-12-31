---
layout: post  
title:  "React和JSX入门指导"  
category: tech  
author: "yinxin630"
---

标签：React JSX

---

原文：[点我打开][1]  
翻译：[碎碎酱][2]  
注：因为原作者使用的React版本较为落后，本人修改了部分内容使其与现在的最新版本相符合。

React是一个用来构建用户页面的开源库，你可以很容易的用它创建视图，并且可以确保使UI与数据模型保持同步。这篇文章是针对初学者的，包含基础的React和JSX语法。 

## 开始使用React

在开始使用之前，首先我们要前往React的[官方网站][3]，下载React Starter Kit(本篇文章使用0.14.5版本)，里面包含着全部你接下来需要用到的库文件和一些官方示例。

如果你下载的是.zip压缩文件，解压它到任意一个目录，你将会看到一个叫做react-x.x.x的目录，在我的机器上是react-0.14.5，打开它并进入build目录，我们将要使用下面两个文件：

`react.js`: React的核心库  
`react-dom.js`: React渲染库，将React组件渲染为真实DOM

在`react-<version>`目录创建一个`index.html`文件，添加下面的代码：

```
<!DOCTYPE html>
<html>
  <head>
    <title>My First React Example</title>
  </head>
  <body>
    <div id="app"></div>
    
    <script src="build/react.js"></script>
    <script src="build/react-dom.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>
    <script type="text/babel">
      var Greeting = React.createClass({
        render: function() {
          return (
            <p>Hello, React</p>
          )
        }
      });
      ReactDOM.render(
        <Greeting/>,
        document.getElementById('app')
      );
    </script>
  </body>
</html>
```

在浏览器中打开它，上面的代码段将会显示`Hello, React`，你需要记住以下几点：

1. React遵循面向对象的思想来开发组件，通常，你会分解你的整个UI为一个组件的集合，在我们的例子中，我们仅仅包含一个叫做Greeting的组件。在React中，你调用`React.createClass()`来创建一个组件，每一个组件都包含一个`render()`函数用来返回将要被渲染的HTML标签，在上面的例子中，我们仅仅简单的返回了`<p>Hello, React</p>`，它将会在视图上显示。

2. 组件在它被渲染之前不会做任何事情，你需要调用`ReactDOM.render()`来渲染组件，并将想要被渲染的组件作为第一个参数，该函数的第二个参数是你想要渲染到的HTML元素，在我们的例子中，我们把Greeting组件渲染到了div#app上面。

3. 你可能会想知道`<Greeting/>`到底是什么，这句代码是JSX（JavaScript XML）语法，它可以使你使用类似HTML的代码来创建DOM节点，并且，是否使用JSX是可选的，但是它具备大量的十分吸引人的特性使你完全没有理由不使用它。

4. React包含ES6语法，浏览器目前不支持全部ES6语法，我们需要首先将其转化为ES5语法，添加下面的语句：
`<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>`

这段代码会在`<script type="text/babel"></script>`标签中查找babel代码，并将其转化为ES5代码。因为React代码需要经过babel转换和JSX转换，这些转换在开发环境时工作的很快，但是当你在生产环境中部署时，需要提前转换你的代码，以便使你的app渲染的更快，我们将会在后面告诉你如何做。

## 使用props

React使用单向的数据流，这意味着数据流仅仅只有一个方向：从父组件到子组件通过属性传递，这些属性通过JSX标签的属性传递，在组件内，你可以通过组件的`props`对象访问传递过来的属性，当属性被更改的时候，React确保你的组件被重新渲染，使你的UI与数据模型保持同步。接下来，我们修改前面的例子使其每1秒显示一个随机的消息，出于精简的代码的考虑，仅仅展示被修改的代码部分：

```
<script type="text/babel">
    var Greeting = React.createClass({
    render: function() {
        return (
        <p>{ this.props.message }</p>
        )
    }
    });
    
    setInterval(function() {
    var messages = ['Hello, World', 'Hello, React', 'Hello, Suisuijiang', 'Hello, Tom'];
    var randomMessage = messages[Math.floor(Math.random() * 4)];
    
    ReactDOM.render(
        <Greeting message={randomMessage}/>,
        document.getElementById('app')
    );
    }, 1000);
</script>
```

上面的代码在数组中随机选择一条消息，每经过1秒就重新渲染一次组件，被选中的消息通过`message`这个属性传递给组件，你需要将变量放在{}里。在Greeting组件中，我们使用`this.props.message`来访问传递过来的消息内容。

如果你尝试运行了上面的代码，它将会每经过1秒输出一条随机的消息内容（假设每次都选取到了不同的消息）。

需要记住，被传递过来的props是不可修改的，你只能通过组件的props传递不同的属性值进来，这样可以保持数据流是单向的，并且它使人清楚的知道你的整个应用是如何随数据改变而变化的。

## 使用state和events

在React里，每个组件都是良好封装的并且独自维护它自身的状态（如果它是有状态的），一个有状态的组件可以在它的state中存储值，并且可以将这个值通过子组件的props传递到子组件中，这可以保证，只要组件的状态发生了改变，子组件的props就会跟着改变，并且，子组件会随着props的改变而自动的被重新渲染。

为了展示这些概念，我们来修改前面的代码，使其当按钮被点击时显示一条随机的消息，为了实现这些，我们定义两个组件：
`RandomMessage`: 产生一条随机消息的父组件
`MessageView`: 处理消息显示的子组件

### RandomMessage

```
<script type="text/babel">
    var MessageView = React.createClass({
    render: function() {
        return (
        <p>{ this.props.message }</p>
        )
    }
    });

    var RandomMessage = React.createClass({
    getInitialState: function() {
        return {message: 'Hello, World'};
    },
    _onClick: function() {
        var messages = ['Hello, World', 'Hello, React', 'Hello, Tom'];
        var randomMessage = messages[Math.floor((Math.random() * 3))];

        this.setState({message: randomMessage});
    },
    render: function() {
        return (
        <div>
            <MessageView message={this.state.message}/>
            <p><input type="button" onClick={this._onClick} value="Change Message"/></p>
        </div>
        )
    }
    });
    
    ReactDOM.render(
    <RandomMessage/>,
    document.getElementById('app')
    );
</script>
```

我们的`RandomMessage`组件在它自身的state中维持一条消息，每一个React组件都包含`getInitialState()`函数用来设置组件的初始化state数据，在我们的例子中，我们使用`Hello, World`来初始化消息。

下一步，我们需要显示一个按钮，当它被点击的时候，使用一个新的message值更新state，我们的组件返回下面的HTML标签：

```
<div>
  <MessageView message={this.state.message}/>
  <p><input type="button" onClick={this._onClick} value="Change Message"/></p>
</div>
```

如你所见，`RandomMessage`组件渲染了另一个组件`MessageView`和一个按钮，注意，组件state中存储的消息是通过props传递到子组件的，我们给按钮添加了一个点击处理事件`this._onClick`，组件会处理这个事件。注意驼峰式命名`onClick`，在HTML中的事件名称都是小写的，比如`onclick`，但是在JSX中你需要大驼峰式的事件名称。

点击事件处理过程选择了一条随机的消息，并通过调用`this.setState()`改变组件的state：

```
this.setState({message:randomMessage})
```

`setState()`是唯一的通知React数据改变的方式，这个函数更新组件当前的state值并重新渲染，被传递过来的props会被重新计算，传递给子组件的props也会改变并且子组件被重新渲染。

### MessageView

```
var MessageView = React.createClass({
  render: function() {
    return (
      <p>{this.props.message}</p>
    )
  }
});
```

这个组件仅仅在UI上显示被传递进来的message属性，你需要知道，它是一个无状态的组件，并且是被有状态的`RandomMessage`组件渲染。

现在，我们已经创建了必须的组件，是时候渲染顶级组件`RandomMessage`了，如下面代码所示：

```
React.render(
  <RandomMessage />,
  document.getElementById('app')
);
```

这就是完整的app，每当你点击按钮时你将会看到一个不同的消息。

## 分离JSX代码

至今为止，我们都是在HTML的script标签中写JSX代码，为了使你的app更加结构化，你应该保持每个组件在他们自己的.jsx文件中，因此我们将上面的代码段放到`random-message.jsx`中，在HTML中使用如下代码引入它：

```
<script type="text/jsx" src="random-message.jsx"></script>
```

因此我们最新的index.html如下所示：

```
<!DOCTYPE html>
<html>
  <head>
    <title>My First React Example</title>
  </head>
  <body>
    <div id="app"></div>
    <script src="build/react.js"></script>
    <script src="build/react-dom.js"></script>
    <script type="text/babel" src="random-message.jsx"></script>
  </body>
</html>
```

## 预编译JSX

我在前面提到的，在生产环境中预编译JSX是很好的做法，为了实现这个，你可以安装一个叫做`react-tools`的node模块（`npm install react-tools -g`），这个工具可以持续的监视你的JSX文件的变动，并且将它们变异成JavaScript，你可以使用下面的命令：

```
jsx   --watch   src/    out/
```

## 关于Virtual DOM

React渲染十分快速，因为它使用了叫做virtual DOM的技术，它在内存中维持一个快速更新的DOM树，并且从不会直接操作真实的DOM，一个组件的`render()`函数调用通知React在当前时间该如何显示DOM。你已经见到了，我们在组件的`render()`函数返回React组件和HTML标签，但是它们不会产生真实DOM，而仅仅是DOM的描述。React会计算这个DOM描述与内存中的DOM树之间的差异，并用一个最快速的方式去更新真实DOM。

## 结束

这篇文章仅仅是React和JSX的一个概述，React还有很多其它的优点，如果你想要查看更多的内容，请前往[React官网][3]。

  [1]: http://www.sitepoint.com/getting-started-react-jsx/
  [2]: http://www.suisuijiang.com
  [3]: https://facebook.github.io/react/