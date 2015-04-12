---
layout: post
title:  "css 实践总结"
category: tech
author: "YshadowZ"
---
近一段时间，和``css``打交道比较多，在此总结一下学习经验，以及实践过程中遇到的一些问题。从基本说起。
***
## 关于盒模型：
首先简述一下最基本的概念。
盒模型，任何标签都是装着内容的盒子。常见有块级元素（浏览器中自己独占一行）和行内元素（自己多大就占多大）。基本设置如下：

* ``width/height``属性，是对盒子中内容尺寸的设置；
* ``padding``属性，是对盒子中内容与其边框线之间的空间的设置；
* ``border``属性，是对边框线的样式的设置；
* ``margin``属性，是对标签与标签之间的间隔的设定；
* ``background``属性，是对边框内的区域（包括``padding``区域在内）的属性的设置；

举例简述最后一条，比如有如下的``<div>``标签（没有设置``height``）：
```
<div class="padding-background"></div>
<style>
	.padding-background{
		padding: 10px;
		background-color: red;
	}
</style>
```
这样，浏览器就会渲染出一个和浏览器等宽，高度为``20px``的矩形。

``border，padding，margin``都可以分别设置``top，right，bottom，left``的值（但是对于行内元素，``padding，margin``只影响左右，不影响上下 [因为行内元素的``line-height``不会变化]，并且``width，height``对其不起作用）。整个盒模型看起来就像是地球的构造，``content``区域是地核，``padding``区域是地幔，``border``是地壳，``margin``是大气层。

这里有一个对块级元素活用``width，height``和``border``的例子：
```
<div class="triangle"></div>
<style>
	.triangle {
		width: 0;
		height: 0;
		border-width: 16px;
		border-style: solid;
		border-color: transparent red transparent transparent;
	}
</style>
```
这样出来的是一个向左的红色三角形。
***
# 关于布局：
浏览器中的元素从左到右，自上到下依次排列。关于元素布局，由此引出的比较重要的属性是``position``和``float``。

<font size='6'>**``position``**</font>，用来对元素进行定位。

* ``static`` 浏览器默认值；
* ``relative`` 相对于元素在文档流中本该出现的位置进行定位；
* ``absolute`` 相对于其第一个被设置过``position``且值不为``static``的父级元素进行定位（采用此属性值的元素会脱离页面的正常的文档排布流程，其他元素若在页面中继续排布，则会无视此元素，由此造成的后果就是元素排布发生重叠）；
* ``fixed`` 相对于浏览器进行定位，如果想在布局中设置一些不随页面滚动而滚动的元素，可以使用此属性值；

关于第三条，比如下面的例子：
```
<div class="position-absolute"></div>
<div class="position-relative"></div>
<style>
	.position-absolute{
		position: absolute;
		width: 100px;
		height: 100px;
		background-color: red;
		margin: 10px;
	}
	.position-relative{
		position: relative;
		width: 100px;
		height: 100px;
		background-color: red;
		margin: 10px;
	}
</style>
```
若果两个元素的``position``都是``relative``， 则每个元素占一行，页面正常排布。如果如上面所示，第一个元素``position``为``absolute``，则后面的元素会无视它的存在，继续出现在第一个元素的位置上（认为此位置没有被占用），造成两个元素重叠。

``position``属性被设定为``relative, absolute``或``fixed``属性中的任何一个后，同时开始生效的有``z-index``和``top，bottom，left，right``属性，即这几个属性依赖于``position``属性。

**这里面有一些细节**，``top，bottom，left，right``都是针对被定位元素的``border``开始进行偏移的，比如下面的例子：
```
<div class="outer">
	<div class="inner"></div>
</div>
<style>
	.outer{
		position: relative;
		width: 100px;
		height: 100px;
		padding: 10px;
	}
	.inner{
		position: absolute;
		left: 10px;
	}
</style>
```
这里``inner``类中的``left``属性设置为``10px``或者不设置，对于``<div class="inner"></div>``的显示是没有影响的，因为它的父级元素的``padding``就是``10px``。

**另一些细节**，关于z-index：
```
<div class="parentOne">
	<div class="oneSon1"></div>
	<div class="oneSon2"></div>
</div>
<div class="parentTwo">
	<div class="twoSon1"></div>
</div>
<style>
	.parentOne{
		position: relative;
		z-index: 1;
		background-color: red;
		width: 100px;
		height: 100px;
	}
	.parentTwo{
		position: absolute;
		z-index: 10;
		background-color: green;
		width: 100px;
		height: 100px;
		top: 10px;
		left: 10px;
	}
	.oneSon1{
		position: absolute;
		z-index: 20;
		background-color: blue;
		width: 100px;
		height: 100px;
		top: 20px;
		left: 20px;
	}
	.oneSon2{
		position: absolute;
		z-index: 30;
		background-color: black;
		width: 100px;
		height: 100px;
	}
	.twoSon1{
		position: absolute;
		z-index: 2;
		background-color: yellow;
		width: 100px;
		height: 100px;
		left: 10px;
		top: 10px;
	}
</style>
```
``parentOne``与``parentTwo``同为父级元素，``z-index``值前者小于后者，``parentOne``中有``oneSon1``和``oneSon2``两个子元素，``z-index``值前者小于后者，但都大于两个父级元素。

真实的覆盖情况是``oneSon2``盖住``oneSon1``盖住``parentOne``，但这三个都被``parentTwo``盖住，然后被``twoSon1``盖住，虽然``parentTwo``，``twoSon1``的``z-index``小于``oneSon1``和``oneSon2``。若去掉``parentOne``中的``z-index``属性，则这五个元素统一按``z-index``值排序。

由此看出：**设置了``z-index``值的元素``A``，如果其没有一个设置过``z-index``属性的父级元素``B``，则它和页面上的其他元素按照``z-index``统一排序，若有，和其父元素同级的其他元素``C``的``z-index``值如果大于其父元素``B``，那么不管这个``A``的``z-index``值是多少，都会被盖在``C``及其子元素下面。**（拼爹的感觉）

<font size='6'>**``float``**</font>，使元素在页面中浮动，用来进行复杂的网页排版，多用于进行多列布局，例子如下：
```
<div class="col-left"></div>
<div class="col-center"></div>
<style>
	.col-left{
		float: left;
		width: 100px;
		height: 100px;
		background-color: red;
	}
	.col-center{
		float: left;
		width: 100px;
		height: 100px;
		background-color: black;
	}
</style>
```
这样就基于浮动构建出了一个两列的布局。

浮动后，如果是块级元素，该元素就不会独占一行，跟在浮动元素后面的元素会上移，包围浮动的元素。例子如下：
```
<div class="float-left"></div>
<div class="float-right"></div>
<p>there is something</p>
<style>
	.float-left{
		float: left;
		width: 100px;
		height: 100px;
		background-color: red;
	}
	.float-right{
		float: right;
		width: 100px;
		height: 100px;
		background-color: red;
	}
</style>
```
``<p>``元素中的内容显示在了两个中间。

但在运用``float``进行实际编程过程中出现过如下问题：
当``<p>``中的内容较多，导致``<p>``的高度超过了两边``<div>``的高度时，``<p>``中超出的那一部分内容会出现在``<div>``的下方，但是我们又希望是一个三列的布局，中间的内容不会溢出到两边。

解决问题的办法：
```
<div class="float-left"></div>
<div class="float-right"></div>
<p>there is something</p>
<style>
	.float-left{
		float: left;
		width: 100px;
		height: 100px;
		background-color: red;	
	}
	.float-right{
		float: right;
		width: 100px;
		height: 100px;
		background-color: red;
	}
	p{
		overflow: hidden;
	}
</style>
```
这样，一个完美的三列布局就出来了。

可能有的同学会问，为什么这三个标签不都是用``float:left``来布局，这样不就是三列了吗？这里就有一个坑，如果三个元素都使用``float:left``来布局，那么必须设定好这三个元素的各自的宽度，否则，页面布局就会乱掉。**每一个浮动后的元素（如果没有设定宽度）都尽可能的占用这一行剩下的空间，然后导致没有地方的元素被挤到下一行**。

导致这个问题的原因是：
**``BFC``**（官方：``block formatting context``，块级格式化上下文，貌似高大上！）

其实就是每一个设置了``float``为非``none``的元素，它自身就触发了``BFC``，触发了``BFC``的元素就是一个独立的盒子，即一个独立的布局空间，元素内部的布局不会受到外部元素的影响，也不会影响到外部元素。

``BFC``中有一条属性叫做：**每一个盒子的左外边缘（``margin-left``）会触碰到容器的左边缘（``border-left``），对于从右到左的格式来说，则触碰到右边缘**。

这句话的意思是你``<p>``标签设置了``margin-left``，我浮动元素浮动到了你``<p>``的内部，但是我自身就是一个``BFC``，我不会受到你``<p>``标签元素属性的影响，所以我不会受``margin-left``影响。所以``<p>``元素过高时，其超过``<div>``的部分的``box margin``的左边与左边``<div>``的``box border``的左边相接触，导致了这种现象。

解决问题的办法也是应用了``BFC``的性质（矛与盾）：**``BFC``的区域不会与``float box``叠加**。

解决问题的关键点在于给p元素设置了``overflow: hidden``属性。先不说这个属性本身的作用是什么，这个属性之所以起作用在于它在这里重新触发了一个新的``BFC``。``BFC``的区域不会和``float box``叠加，因此问题的到了解决。

``BFC``更详细的属性和用法可参考此[网址](http://www.html-js.com/article/1866)。

说到**布局**，与其相关的属性比较常用的还有：

* ``max(min)-height`` 设置高度的上限（下限）
* ``max(min)-width`` 设置宽度的上限（下限）
* ``overflow`` 元素内部的内容要溢出时所采取的行为
* ``display`` 设置元素类型
* ``visibility`` 设置元素的可见性
* ``line-height`` 设置元素的行高
* ``text-align`` 设置元素中的内容的对其方式

其中，``display: none``与``visibility: hidden``的区别是，**前者元素及其所占的位置都消失，深藏功与名**；后者仅仅是"在线对其隐身——假离线"，**元素隐藏，但其依旧在文档流中占据一席之地**，往往会在正常页面中留下窟窿。不过``visibility``属性经常用在脱离了文档流的绝对定位元素身上，反正绝对定位的元素不在正常的次元里面。

``line-height``属性用来设置行高。曾经在基于``bootstrap``做一个返回顶部的按钮时，被此属性干扰过，详情：
```
<div class="to-top">
	<span class="fa fa-arrow-up"></span>
</div>
<style>
	.to-top{
		width: 40px;
		height: 40px;
		background-color: blue;
	}
</style>
```
这样出来的按钮不是正方形，而是一个长方形，百思不得解。后来找到了原因，``bootstrap``中对``<div>``有默认的``line-height``，从而导致了``<div>``高度小于``line-height``时，被其内部的元素撑开。

辩证的看待``line-height``也有好用的一面，可以用来让内容垂直居中，如下：
```
<p class="sentence">there are some words</p>
<style>
	.sentence{
		height: 40px;
		line-height: 40px;
		background-color: red;
	}
</style>
```
除此，若想让某元素中的内容垂直居中，也可以通过``display``与``vertical-align``属性搭配，如下：
```
<p class="sentence">there are some words</p>
<style>
	.sentence{
		height: 40px;
		display: table-cell;
		vertical-align: middle;
	}
</style>
```
这样的话，``<p>``元素就不再是一个块级元素，而是变成了表格单元格。

也可以这样，对于一个块级元素，不设置其高度，只设置其``padding``，这样其内部的内容看上去就是垂直居中的：
```
<div class="outer">
	something
</div>
<style>
	.outer{
		padding: 10px;
		background-color: red;
	}
</style>
```
***
# 关于其他常用``css``属性：

* ``color`` 设定文本颜色，可继承，
* ``font-size`` 设定字号大小，可继承
* ``box-shadow`` 给元素添加阴影
* ``border-radius`` 给元素设置圆角
* ``background-color`` 设定背景颜色
* ``clear`` 清除浮动
* ``cursor`` 设置鼠标图形
* ``opacity`` 设置元素透明度

此外还有一些炫酷的``css3``属性，如动画，渐变，更多的布局属性，这些都是后话了。
***


# 关于``css``结构重构总结：

* **开始，工程中的``css``结构是这样的：**
``bootstrap``是基础，最底层，提供基本的布局模式及一些常用样式，向上是``flat-ui``（基于``bootstrap``进行了美化），向上是针对网站修改后的全站通用的自定制版的``flat-ui``，向上是针对各个``app``的具体的``css``样式。

* **重构之后的``css``结构如下：**
``bootstrap``是基础，向上是针对网站做的一些``reset``（即一些属性的默认值修改），向上是从``flat-ui``中抽出的公用样式，向上是各个``app``或者各个组件的``css``样式。

* **命名：**
为防止全局``css``名称冲突，各个``app``或者各个组件的``css``样式命名，都加上了前缀（即``app``或组件名称）或者命名空间，公用``css``命名前也添加了统一前缀。

***

# 关于编写``css``样式：

* 尽量使用``css``强大的选择器，这样有助于减少过多的类的命名与书写，同时增强模块的内聚程度。

* 使用一种``css``预处理框架，``less``或者``sass``，这样有助于提高``css``的工程性，使其结构看上去不那么散乱，同时也提高了编写``css``的严谨性与开发速度，减少编写的代码量。
***

# 写在最后（网站推荐）：

* **眼界大开脑洞：**
[炫酷的前沿``css``设计](http://tympanus.net/)

* **深入学习css**
[关于``display``](http://www.cnblogs.com/StormSpirit/archive/2012/10/17/2726994.html)
[布局](http://zh.learnlayout.com/)

* **学习sass：** 
[阮老师的入门讲解](http://www.ruanyifeng.com/blog/2012/06/sass.html)
[中文](http://www.w3cplus.com/sassguide/)
[官方文档](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)

* **学习less：**
[官方文档](http://less.bootcss.com/)