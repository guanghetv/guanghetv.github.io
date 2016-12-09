---
layout: post  
title:  "BEM核心概念"  
category: news  
author: "yinxin630"
---

原文：[点击打开][1]  
翻译：[碎碎酱][2]

## Block

Block是一个在逻辑上和功能上独立的页面组件，等价于Web Components标准中的组件，一个Block就是行为（JavaScript）、模板、风格（CSS）和[其它技术][3]和合集，独立的Block使得可以被重用，并且可以促进项目开发效率以及节约项目维护成本。

### Block特性

#### 结构嵌套

一个Block组件可以被嵌套在任意其它组件中。

例如，一个`head`可以包含一个logo（`logo`）、一个搜索框（`search`）和一个登录组件（`auth`）。

![此处输入图片的描述][4]

#### 任意定位

Block可以在页面内任意移动，在项目的不同页面中复用，Block被设计为一个独立的实体，使得可以修改它在页面中的位置，并且不会影响它的功能和外观。

比如，logo和登录框的位置可以互相调换，而且你不需要修改任何任何Block的JavaScript和CSS代码。

![此处输入图片的描述][5]
![此处输入图片的描述][6]

#### 复用

一个界面可以包含多个相同的Block实例。

![此处输入图片的描述][7]

## Element

Block中的任一组成部分，都不能够在它之外使用。

比如，一个菜单项不能走菜单Block之外使用，因为它是一个Element。

![此处输入图片的描述][8]

[A block or an element: when should I use which?][9]
[Using elements within elements is not recommended by the BEM methodology][10]

## Modifier

一个BME实体定义了一个Block或者Element的行为和外观。

是否使用Modifier是可选的。

Modifier在本质上和HTML的属性是类似的，相同的Block会因为使用不同的Modifier而看起来不同。

例如，菜单Block的外观会依赖于所使用的Modifier。

![此处输入图片的描述][11]

## BEM entity

Block、Element和Modifier都称为BEM entity。

这个概念既可以应用与单个的BEM实体，也可以作为blocks、elements和modifiers的通称。

## Mix

每个不同的BEM实体都被托管在一个DOM节点上。

Mixes允许这么做：

* 组合多个BEM实体的行为和外观，避免重复代码
* 在已有BEM实体的基础上创建新的语义化的接口组件

我们来考虑下这种情况，mix包含一个Block和和另一个Block元素。

我们假设它通过一个`link` Block连接到你的项目中，我们需要把菜单项格式化为链接，下面有多个可行办法：

* 为菜单项创建Modifier来使其指向一个链接，实现这样的Modifier必然会复制`link` Block的行为和外观，这会导致代码复制。
* 创建一个mix组合`link`组件，并把`lick`组件作为`menu`的Element，混合两个BEM实体可以让我们使用`lick`组件的基础功能，并为`menu`添加额外的CSS规则而不需要复制任何代码。

## BEM树

我们使用Blocks、Elements和Modifiers来表示一个web页面的结构，用BEM实体、their states、order、nesting、和辅助数据来抽象的描述DOM树。

在真实的项目中，一个BEM树可以用多种格式化数据描述其结构。

我们来看看这样一个DOM树：

```
<header class="header">
    <img class="logo">
    <form class="search-form">
        <input type="input">
        <button type="button"></button>
    </form>
    <div class="lang-switcher"></div>
</header>
```

相等的BEM树会是这样的：

```
header
    ├──logo
    └──search-form
        ├──input
        └──button
    └──lang-switcher
```

使用XML和[BEMJSON][12]来表示：

XML

```
<block:header>
    <block:logo/>
    <block:search-form>
        <block:input/>
        <block:button/>
    </block:search-form>
    <block:lang-switcher/>
</block:header>
```

BEMJSON

```
{
    block: 'header',
    content : [
        { block : 'logo' },
        {
            block : 'search-form',
            content : [
                { block : 'input' },
                { block : 'button' }
            ]
        },
        { block : 'lang-switcher' }
    ]
}
```

## Block实现

一些不同[技术][13]共同定义了如下的BEM实体

* behavior
* appearance
* tests
* templates
* documentation
* description of dependencies
* additional data (e.g., images).

## 技术实现

Block可以使用一个或多个技术实现，例如：

* behavior — JavaScript, CoffeeScript
* appearance — CSS, Stylus, Sass
* templates — BEMHTML, BH, Jade, Handlebars, XSL
* documentation — Markdown, Wiki, XML.

举个例子，如果一个Block的外观是用CSS定义的，这就意味着这个Block是用CSS技术实现的，同样的，如果一个Block的文档是用Markdown格式写的，那这个文档就是用Markdown技术实现的。

## Redefinition level

A set of BEM entities and their partial implementations.

The final implementation of a block can be divided into different redefinition levels. Each new level extends or overrides the original implementation of the block. The end result is assembled from individual implementation technologies of the block from all redefinition levels in a pre-determined consecutive order.

Any implementation technologies of BEM entities can be redefined.

For example, there is a third-party library linked to a project on a separate level. The library contains ready-made block implementations. The project-specific blocks are stored on a different redefinition level.

Let's say we need to modify the appearance of one of the library blocks. That doesn't require changing the CSS rules of the block in the library source code or copying the code at the project level. We only need to create additional CSS rules for that block at the project level. During the build process, the resulting implementation will incorporate both the original rules from the library level and the new styles from the project level.

  [1]: https://en.bem.info/method/key-concepts/
  [2]: http://www.suisuijiang.com
  [3]: https://en.bem.info/method/key-concepts/#implementation-technology
  [4]: https://img-fotki.yandex.ru/get/15534/158800653.0/0_111fb2_7710ab3d_orig
  [5]: https://img-fotki.yandex.ru/get/16156/158800653.0/0_111fb3_2fec3fed_orig
  [6]: https://img-fotki.yandex.ru/get/15542/158800653.0/0_111fb1_bcbc3c6a_orig
  [7]: https://img-fotki.yandex.ru/get/15498/158800653.0/0_111fb0_fbb195e9_orig
  [8]: https://img-fotki.yandex.ru/get/15588/158800653.0/0_111fb6_192672cf_orig
  [9]: https://en.bem.info/faq#a-block-or-an-element-when-should-i-use-which
  [10]: https://en.bem.info/faq#why-does-bem-not-recommend-using-elements-within-elements-block__elem1__elem2
  [11]: https://img-fotki.yandex.ru/get/16183/158800653.0/0_111fba_921b3c47_orig
  [12]: https://en.bem.info/technology/bemjson/
  [13]: https://en.bem.info/method/key-concepts/#implementation-technologyba_921b3c47_orig
  [12]: https://en.bem.info/technology/bemjson/