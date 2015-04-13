---
layout: post
title:  "css 实践总结"
category: tech
author: "YshadowZ"
---
近一段时间，和css打交道比较多，在此总结一下学习经验，以及实践过程中遇到的一些问题。从基本说起。

***

## 关于盒模型：
首先简述一下最基本的概念。
盒模型，任何标签都是装着内容的盒子。常见有块级元素（浏览器中自己独占一行）和行内元素（自己多大就占多大）。基本设置如下：

* width/height 是对盒子中内容尺寸的设置；
* padding 是对盒子中内容与其边框线之间的空间的设置；
* border 是对边框线的样式的设置；
* margin 是对标签与标签之间的间隔的设定；
* background 是对边框内的区域（包括padding区域在内）的属性的设置；

举例简述最后一条，比如有如下的<div>标签（没有设置height）：
````
<div class="padding-background"></div>
<style>
	.padding-background{
		padding: 10px;
		background-color: red;
	}
</style>
````
这样，浏览器就会渲染出一个和浏览器等宽，高度为20px的矩形。

border，padding，margin都可以分别设置top，right，bottom，left的值（但是对于行内元素，padding，margin只影响左右，不影响上下 [因为行内元素的line-height不会变化]，并且width，height对其不起作用）。整个盒模型看起来就像是地球的构造，content区域是地核，padding区域是地幔，border是地壳，margin是大气层。

这里有一个对块级元素活用width，height和border的例子：
````
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
````
这样出来的是一个向左的红色三角形。

***

## 关于布局：
浏览器中的元素从左到右，自上到下依次排列。关于元素布局，由此引出的比较重要的属性是position和float。

<font size='5'>**position**</font>，用来对元素进行定位。

* static 浏览器默认值；
* relative 相对于元素在文档流中本该出现的位置进行定位；
* absolute 相对于其第一个被设置过position且值不为static的父级元素进行定位（采用此属性值的元素会脱离页面的正常的文档排布流程，其他元素若在页面中继续排布，则会无视此元素，由此造成的后果就是元素排布发生重叠）；
* fixed 相对于浏览器进行定位，如果想在布局中设置一些不随页面滚动而滚动的元素，可以使用此属性值；

关于第三条，比如下面的例子：
````
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
````
若果两个元素的position都是relative， 则每个元素占一行，页面正常排布。如果如上面所示，第一个元素position为absolute，则后面的元素会无视它的存在，继续出现在第一个元素的位置上（认为此位置没有被占用），造成两个元素重叠。

position属性被设定为relative, absolute或fixed属性中的任何一个后，同时开始生效的有z-index和top，bottom，left，right属性，即这几个属性依赖于position属性。

**这里面有一些细节**，top，bottom，left，right都是针对被定位元素的border开始进行偏移的，比如下面的例子：
````
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
````
这里inner类中的left属性设置为10px或者不设置，对于<div class="inner"></div>的显示是没有影响的，因为它的父级元素的padding就是10px。

**另一些细节**，关于z-index：
````
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
````
parentOne与parentTwo同为父级元素，z-index值前者小于后者，parentOne中有oneSon1和oneSon2两个子元素，z-index值前者小于后者，但都大于两个父级元素。

真实的覆盖情况是oneSon2盖住oneSon1盖住parentOne，但这三个都被parentTwo盖住，然后被twoSon1盖住，虽然parentTwo，twoSon1的z-index小于oneSon1和oneSon2。若去掉parentOne中的z-index属性，则这五个元素统一按z-index值排序。

由此看出：**设置了z-index值的元素A，如果其没有一个设置过z-index属性的父级元素B，则它和页面上的其他元素按照z-index统一排序，若有，和其父元素同级的其他元素C的z-index值如果大于其父元素B，那么不管这个A的z-index值是多少，都会被盖在C及其子元素下面。**（拼爹的感觉）

<font size='5'>**float**</font>，使元素在页面中浮动，用来进行复杂的网页排版，多用于进行多列布局，例子如下：
````
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
````
这样就基于浮动构建出了一个两列的布局。

浮动后，如果是块级元素，该元素就不会独占一行，跟在浮动元素后面的元素会上移，包围浮动的元素。例子如下：
````
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
````
<p>元素中的内容显示在了两个中间。

但在运用float进行实际编程过程中出现过如下问题：
当<p>中的内容较多，导致<p>的高度超过了两边<div>的高度时，<p>中超出的那一部分内容会出现在<div>的下方，但是我们又希望是一个三列的布局，中间的内容不会溢出到两边。

解决问题的办法：
````
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
````
这样，一个完美的三列布局就出来了。

可能有的同学会问，为什么这三个标签不都是用float:left来布局，这样不就是三列了吗？这里就有一个坑，如果三个元素都使用float:left来布局，那么必须设定好这三个元素的各自的宽度，否则，页面布局就会乱掉。**每一个浮动后的元素（如果没有设定宽度）都尽可能的占用这一行剩下的空间，然后导致没有地方的元素被挤到下一行**。

导致这个问题的原因是：
**BFC**（官方：block formatting context，块级格式化上下文，貌似高大上！）

其实就是每一个设置了float为非none的元素，它自身就触发了BFC，触发了BFC的元素就是一个独立的盒子，即一个独立的布局空间，元素内部的布局不会受到外部元素的影响，也不会影响到外部元素。

BFC中有一条属性叫做：**每一个盒子的左外边缘（margin-left）会触碰到容器的左边缘（border-left），对于从右到左的格式来说，则触碰到右边缘**。

这句话的意思是你<p>标签设置了margin-left，我浮动元素浮动到了你<p>的内部，但是我自身就是一个BFC，我不会受到你<p>标签元素属性的影响，所以我不会受margin-left影响。所以<p>元素过高时，其超过<div>的部分的box margin的左边与左边<div>的box border的左边相接触，导致了这种现象。

解决问题的办法也是应用了BFC的性质（矛与盾）：**BFC的区域不会与float box叠加**。

解决问题的关键点在于给p元素设置了overflow: hidden属性。先不说这个属性本身的作用是什么，这个属性之所以起作用在于它在这里重新触发了一个新的BFC。BFC的区域不会和float box叠加，因此问题的到了解决。

BFC更详细的属性和用法可参考此[网址](http://www.html-js.com/article/1866)。

说到**布局**，与其相关的属性比较常用的还有：

* max(min)-height 设置高度的上限（下限）
* max(min)-width 设置宽度的上限（下限）
* overflow 元素内部的内容要溢出时所采取的行为
* display 设置元素类型
* visibility 设置元素的可见性
* line-height 设置元素的行高
* text-align 设置元素中的内容的对其方式

其中，display: none与visibility: hidden的区别是，**前者元素及其所占的位置都消失，深藏功与名**；后者仅仅是"在线对其隐身——假离线"，**元素隐藏，但其依旧在文档流中占据一席之地**，往往会在正常页面中留下窟窿。不过visibility属性经常用在脱离了文档流的绝对定位元素身上，反正绝对定位的元素不在正常的次元里面。

line-height属性用来设置行高。曾经在基于bootstrap做一个返回顶部的按钮时，被此属性干扰过，详情：
````
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
````
这样出来的按钮不是正方形，而是一个长方形，百思不得解。后来找到了原因，bootstrap中对<div>有默认的line-height，从而导致了<div>高度小于line-height时，被其内部的元素撑开。

辩证的看待line-height也有好用的一面，可以用来让内容垂直居中，如下：
````
<p class="sentence">there are some words</p>
<style>
	.sentence{
		height: 40px;
		line-height: 40px;
		background-color: red;
	}
</style>
````
除此，若想让某元素中的内容垂直居中，也可以通过display与vertical-align属性搭配，如下：
````
<p class="sentence">there are some words</p>
<style>
	.sentence{
		height: 40px;
		display: table-cell;
		vertical-align: middle;
	}
</style>
````
这样的话，<p>元素就不再是一个块级元素，而是变成了表格单元格。

也可以这样，对于一个块级元素，不设置其高度，只设置其padding，这样其内部的内容看上去就是垂直居中的：
````
<div class="outer">
	something
</div>
<style>
	.outer{
		padding: 10px;
		background-color: red;
	}
</style>
````
***

## 关于其他常用css属性：

* color 设定文本颜色，可继承，
* font-size 设定字号大小，可继承
* box-shadow 给元素添加阴影
* border-radius 给元素设置圆角
* background-color 设定背景颜色
* clear 清除浮动
* cursor 设置鼠标图形
* opacity 设置元素透明度

此外还有一些炫酷的css3属性，如动画，渐变，更多的布局属性，这些都是后话了。

***


## 关于css结构重构总结：

* **开始，工程中的css结构是这样的：**
bootstrap是基础，最底层，提供基本的布局模式及一些常用样式，向上是flat-ui（基于bootstrap进行了美化），向上是针对网站修改后的全站通用的自定制版的flat-ui，向上是针对各个app的具体的css样式。

* **重构之后的css结构如下：**
bootstrap是基础，向上是针对网站做的一些reset（即一些属性的默认值修改），向上是从flat-ui中抽出的公用样式，向上是各个app或者各个组件的css样式。

* **命名：**
为防止全局css名称冲突，各个app或者各个组件的css样式命名，都加上了前缀（即app或组件名称）或者命名空间，公用css命名前也添加了统一前缀。

***

## 关于编写css样式：

* 尽量使用css强大的选择器，这样有助于减少过多的类的命名与书写，同时增强模块的内聚程度。

* 使用一种css预处理框架，less或者sass，这样有助于提高css的工程性，使其结构看上去不那么散乱，同时也提高了编写css的严谨性与开发速度，减少编写的代码量。

***

## 写在最后（网站推荐）：

* **眼界大开脑洞：**
[炫酷的前沿css设计](http://tympanus.net/)
* **深入学习css**
[关于display](http://www.cnblogs.com/StormSpirit/archive/2012/10/17/2726994.html)
[布局](http://zh.learnlayout.com/)
* **学习sass：** 
[阮老师的入门讲解](http://www.ruanyifeng.com/blog/2012/06/sass.html)
[中文](http://www.w3cplus.com/sassguide/)
[官方文档](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)
* **学习less：**
[官方文档](http://less.bootcss.com/)