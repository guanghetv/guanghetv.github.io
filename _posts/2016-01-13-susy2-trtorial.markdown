---
layout: post  
title:  "Susy2教程"  
category: news  
author: "yinxin630"
---

作者：[碎碎酱][1]  
完整代码下载：[GitHub][2]  

Susy是一个创建高度订制grid的实用工具，Susy2已经正式发布了，如果你喜爱Susy1，那么你一定也会喜欢上Susy2，因为它提供了更多的灵活性。

本教程分为两部分，在教程中，我将分享如何使用Susy2创建Complex AG Grid。

## 为什么使用Susy

就像我上面提到的，Susy使你可以轻松的创建各种类型的grid，并且不需要大量的数学计算，Susy的美丽之处在于，它从HTML中分离出了视图的CSS。

如果你在之前使用过传统的grid框架，比如Foundation和Bootstrap，你肯定已经知道，这些框架的grid都有着固定的宽度和视图，而且，如果你想更改你的布局，你必须在HTML代码中添加新的class。

Susy完全抛弃了这种布局方式，你可以只关注CSS类的目的，然后给它使用一个grid。

当然，我知道这说起来很抽象、很难理解，与其一直在这讲原理，如果我们马上开始使用Susy2来构建一个复杂的grid系统，采用Arnaud Guera (AG)设计，包含10个栏目，这个grid看起来就像这样：

![此处输入图片的描述][3]

## 安装Susy2

### 如果你没有安装过Susy2

Susy需要依赖Sass，你可以使用下面的命令安装：

```
$ sudo gem install sass 
$ sudo gem install susy
```

### 如果你已经安装了Susy2

如果你已经安装了Susy2，并且在你的环境中有Ruby RVM，你可以使用gem更新：

```
$ sudo gem update 
```

如果不成功的话，你可以尝试下一个方法，或者先[安装Ruby RVM][4]。

第二种安装Susy2的方法要手动进行，首先卸载上面的两个gem包，重新安装它们，不使用更新来安装的话可以更好的避免错误。

```
$ sudo gem install susy 
$ sudo gem install sass
or
$ sudo gem install sass 
$ sudo gem install susy
```

现在你已经安装了Susy2，我们开始配置Susy2。

## 配置Susy2

如同原来的版本，使用Susy2，需要在config.rb文件中调用susy。

```
# config.rb 
require 'susy'
```

然后你需要在样式表中导入并使用susy。

```
// Importing Susy 
@import 'susy'; 
```

Susy2有大量的[默认全局配置][5]，你可以修改这些配置：

```
// Configuring Susy 2 Global Defaults
$susy: (
  key : value
);
```

你可以使用默认配置，或者修改配置，这取决于你想要如何使用Susy，我将会在另一篇教程中更深入的介绍它的默认配置。你现在可以立即使用默认配置的Susy，但是因为我喜爱使用border-box和rem单位，所以我要修改一些默认配置：

```
$susy: (
  global-box-sizing: border-box,
  use-custom: (rem: true
    )
);
```

请记住，Susy默认是流式布局，这意味着外面的容器元素的宽度是100%的。

如果你想要使用Susy的固定宽度和视图，修改`math`为`static`，这两种模式主要的不同会在另一片教程中讲解。

还有，你必须在项目中包含compass，初始配置看起来大致是这样的：

```
@import "compass";
@import "susy";

// Configuring Susy Defaults
$susy: (
  global-box-sizing: border-box,
  use-custom: (rem: true
    )
  );

@include border-box-sizing;
```

## AG Grid的HTML和CSS代码

我已经在[Susy1教程][6]中创建了同样的AG Grid，所以HTML代码跟原来是一样的。

```
<div class="container">
<h1>The 10 column complex nested grid AG test</h1>

<div class="ag ag1">
  <h2>AG 1</h2>
</div>
<!-- /ag1 -->

<!-- ag4 to ag7 are nested within ag2.-->
<div class="ag ag2">
  <h2>AG 2</h2>
  <div class="ag ag4">
    <h2>AG 4</h2>
  </div>
  <div class="ag ag5">
    <h2>AG 5</h2>
  </div>
  <div class="ag ag6">
    <h2>AG 6</h2>
  </div>

  <!-- ag8, ag9 and ag10 are nested within ag7 -->
  <div class="ag ag7">
    <h2>AG 7</h2>
    <div class="ag ag8">
      <h2>AG 8</h2>
    </div>
    <div class="ag ag9">
      <h2>AG 9</h2>
    </div>
    <div class="ag ag10">
      <h2>AG 10</h2>
    </div>
  </div>
  <!-- /ag7 -->
</div>
<!-- /ag2 -->

<div class="ag ag3">
  <h2>AG 3</h2>
</div>
<!-- /ag3 -->

</div>
<!-- /container -->
```

简单讲，如果内容是在另一个div里面的，你应该在上一个div里面嵌套它。

举个例子，AG4到AG7会嵌套在AG2里面，AG8、AG9和AG10会嵌套在AG10里面。

grid background的CSS代码也是一样的。

```
/**
 * Styles for AG grids & Container
 */

.container {
  background-color: #fbeecb;
}

.ag1, .ag3 {
  background-color: #71dad2;
}

.ag2 {
  background-color: #fae7b3;
}

.ag4,.ag5,.ag8,.ag9 {
  background-color: #ee9e9c;
}

.ag6 {
  background-color: #f09671;
}

.ag7 {
  background-color: #f6d784;
}

.ag10 {
  background-color: #ea9fc3;
}

/**
 * Text Styles
 */
h2 {
  text-align: center;
  font-size: 1rem;
  font-weight: normal;
  padding-top: 1.8rem;
  padding-bottom: 1.8rem;
}
```

好了，现在我们可以投身于更多的Susy的细节。

## 导入Susy2 Mixin和函数

在正式使用Susy2完成grid之前，我们需要知道三个非常重要的被用于Susy的Mixin和函数，如果你明白了这些内容，你就可以使用Susy做任何事情。

### The Container Mixin

首先，你需要给Susy定义一个容器，来使用它的神奇的计算能力。

```
// The Container Mixin 
@include container( [<length>] ); 

// Note: Optional arguments are enclosed in []
```

container mixin有一个可选的`length`参数，允许你设置容器的最大宽度，如果没有这个值，Susy默认的使用`max-width: 100%;`代替。

如果你是使用静态grid，Susy建议让Susy自动为你计算最大宽度。

为了使用自动计算，你可以省略`length`参数。

```
@include container;
```

### Span (Mixin)

span mixin是Susy布局的核心，[Susy使它非常灵活][7]。

我通常使用Susy的方式来写我的布局。

```
@include span( <width> [<wide / wider>] of <layout> [<last>] ); 
```

如果你使用Susy1，你将会注意到这和`span-column` Mixin十分相像，仅有一点点区别。

让我来打破它，详细的解释下这里发生了什么：

* `<width>` refers to the number of columns the element going to take up.
* [`<wide / wider>`]is a optional argument lets you expand the width of the column to include 1 or 2 more gutters respectively
of is a keyword to let Susy know that the layout is coming up.
* `<layout>` is the context of the container, along with other optionals that define the layout. (Context refers to the number of columns the parent container is made up of).
* [`<last>`] is an optional argument that tells Susy that this element is supposed to be the last item in the row.

请看这个例子：

```
// This has a width of 3 columns + 1 gutter, out of 9 columns and is supposed to be the last item within the context. 
.some-selector {
  @include span(3 wide of 9 last); 
}
```

### Span (Function)

Span Function与Span Mixin相似，唯一例外的是它返回元素的宽度，在这你仅仅需要`<width>`、`<wide/wider>`和`<layout>`。

```
// This has a width of 3 columns out of 9 columns
.some-selector {
  width: span(3 of 9);
}
```

这个函数使我们不需要像以前那样使用margin和padding，而是更容易的实现对齐元素，现在你可以仅仅使用Span Function来在Pad上获得正确的宽度。

### Gutter Function

Gutter Function仅关注一个问题 - 上下文环境。

```
// This outputs margin that equals to 1 gutter width in a 9 column layout
.some-selector {
  margin-right: gutter(9); 
}
```

在使用Sysu2时，还有很多重要的函数需要你了解。

## 使用Sysu2构建AG Grid

使用Sysu的第一件事就是创建容器，在这个例子中我们的容易是`.container`。

```
.container {
  @include container;
}
```

我们已经知道了容器会控制浮动的元素，我们将要给它清除浮动。

```
.container {
  // This is the default clearfix from Compass. You can opt to use other clearfix methods
  @include clearfix; 
}
```

首先，我们在AG1、AG2和AG3上创建布局，经过考虑，我们的容器大概需要占据10个栏目，AG1和AG3每个占2个栏目，剩余的被AG2占用，AG2需要清除浮动，并作为AG4到AG10的容器。

```
.ag1 {
  @include span(2 of 10);
}

.ag2 {
  @include span(6 of 10);
  @include clearfix; 
}

.ag3 {
  @include span(2 of 10 last);
}
```

如果你现在看一下输出，它大概是这样的：

![此处输入图片的描述][8]

AG4和AG5嵌套在AG2里面，它们每个占3个栏目。

```
.ag4 {
  @include span(3 of 6);
}

.ag5 {
  @include span(3 of 6 last);
}

Alternatively, you can make use of the last mixin and write it this way. The last mixin just changes that element to be the last in the row. 

.ag4,.ag5 {
  @include span(3 of 6);
}
.ag5 {
  @include last;
}
```

AG4和AG5漂亮的到位了。

![此处输入图片的描述][9]

AG6占2个栏目，AG7占4个栏目，并且AG7是这一行的最后一个项目，或许现在你已经知道该怎么做了。

```
.ag6 {
  @include span(2 of 6);
}

.ag7 {
  @include span(4 of 6 last);
  @include clearfix;
}
```

![此处输入图片的描述][10]

最后，让我们完成AG Grid的最后一小部分。

```
.ag8 {
  @include span(2 of 4);
}

.ag9 {
  @include span(2 of 4 last);
}

.ag10 {
  clear: both;
  // Alternatively, you can now use the span mixin with the full keyword to tell Susy this should be a 100% width 
  // @include span(full);
}
```

![此处输入图片的描述][11]


  [1]: http://www.suisuijiang.com
  [2]: https://github.com/yinxin630/susy2-demo
  [3]: http://www.zell-weekeat.com/wp-content/uploads/2014/04/susy2-1.png
  [4]: http://rvm.io/rvm/install
  [5]: http://susydocs.oddbird.net/en/latest/settings/#global-defaults
  [6]: http://www.zell-weekeat.com/a-complete-tutorial-to-susy/
  [7]: http://susydocs.oddbird.net/en/latest/toolkit/#span-mixin
  [8]: http://www.zell-weekeat.com/wp-content/uploads/2014/04/susy2-2.png
  [9]: http://www.zell-weekeat.com/wp-content/uploads/2014/04/susy2-3.png
  [10]: http://www.zell-weekeat.com/wp-content/uploads/2014/04/susy2-4.png
  [11]: http://www.zell-weekeat.com/wp-content/uploads/2014/04/susy2-1.png