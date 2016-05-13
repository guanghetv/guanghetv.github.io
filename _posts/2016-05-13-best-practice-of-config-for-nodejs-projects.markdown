---
layout: post
title:  "Nodejs项目配置文件最佳实践"
category: tech
author: "libook"
---

# 前言

本文为抛砖引玉。

在项目中使用配置文件有很多好处，
比如减少冗余代码、增强可定制性、消除[魔术数字](https://en.wikipedia.org/wiki/Magic_number_(programming))(提高代码可读性)等等，
那在Nodejs项目中如何使用配置文件比较好呢？

# 需求

在Nodejs项目中，我们对配置文件的需求基本如下：

0. 易于使用；
0. 易于编写和修改；
0. 结构清晰，可读性高；
0. 模板化，可针对不同的运行场景增量合并；
0. 静态化，不参与业务逻辑的状态变化；

# 设计

## 易于使用

通常对配置文件的设计是写成一个，然后在任何使用到配置项的地方
都使用require来引入这个配置文件，缺点是几乎每一个代码文件里都要写一遍require。

其实配置项的应用场景本身就是具有全局性的，所以我们完全可以将配置项作为全局变量
一次引入多次使用。

Nodejs中的全局变量的使用方式是直接将变量声明为global对象的成员：

```javascript
global.a = "This is a global variable.";
```

可以在声明后的整个程序中调用到：

```javascript
console.log(a);// Output => "This is a global variable."

// Or the full version.
console.log(global.a);// Output => "This is a global variable."
```

## 易于编写和修改

定义配置文件可以使用很多方式，XML、YAML、JSON、FUNCTION等等，
对于javascript项目来说，最方便的还是使用JSON来作为配置文件的风格。

## 结构清晰，可读性高

使用JSON除了易于编写和修改以外，还可以一定程度上让配置文件结构清晰、可读性高。

除此之外，最好将所有配置项目按照面向对象的思想进行分类和结构化，这样便于理解，
同时也可以使其更易于编写和修改。

## 模板化，可针对不同的运行场景增量合并

程序通常有多个运行场景，比如开发、测试、生产等等，就会有一些配置项
是随着运行场景的不同而不同的，比如数据库地址、服务端口号等等；而且这些变化的配置项
是少数的几个配置项，不值得为每一个场景都单独写一份配置文件。

所以我们需要一种机制来将配制文件模板化，制定一个默认模板，然后再根据不同的运行场景
来定制个别的配置项。

这种模板化机制实现的关键问题就是如何将一个JSON合并到另一个JSON上去，
方法是使用ES6原生的Object.assign或者直接遍历对象中的所有key，但上述方法只能
作用于Object的第一代Children，所以要用递归的方式进行深度合并。

[node-extend](https://github.com/justmoon/node-extend)很好地
实现了这些功能，使用非常简单：

```javascript
let a = {"x": 1, "z": {"m": 4}},
    b = {"x": 3, "y": "haha"},
    c = {"z": {"m": 9, "n": "aha"}};
extend(true, a, b, c);// Output => { x: 3, z: { m: 9, n: 'aha' }, y: 'haha' }
```

## 静态化，不参与业务逻辑的状态变化

通常配置项是作为变量（逻辑上的常量）来进行调用的，而且这些变量的作用域
都是很广泛的。那么问题来了，变量的值可以被改变，这种改变往往是不经意的，如下：

```javascript
// const a = {
//     "b": {
//         "c": 1
//     }
// };

let x = a.b;
x.c = 4;
console.log(JSON.stringify(a));// Output => {"b":{"c":4}}
```

假设a就是配置对象，那么在类似上述这种“不经意”的使用方法就会“不经意”地改变整个
配置项的值。

为了避免这种事情的发生，我们需要将配置项静态化。

需要说明的是，虽然ES6提供了const指令可以用来声明静态变量，但是const的功能
是极其有限的。const只能锁定变量的引用，对于数值、字符串、布尔值这些普通类型来说
是非常好用的，但是如果变量引用的值是一个对象的话，那么对象的成员也是可以被改变的，
这一点在上一个例子中也有体现。

还有，就是使用const声明的对象依然可以添加新的成员：

```javascript
const a = {"x": 1};
console.log(a);// Output => {x: 1}
a.y = 2;
console.log(a);// Output => {x: 1, y: 2}
```

ES6提供了Object.freeze()，可以将对象“冻结”，冻结的效果包括不能再添加新的对象，
虽然很实用，但依然只能对Object的第一代Children有效。

所以我们需要将所有配置项都进行深度静态化。

这方面有一个很好的工具是[deep-freeze-strict](https://github.com/jsdf/deep-freeze)，
这货有个上游项目是[deep-freeze](https://github.com/substack/deep-freeze)，
前者修复了后者对严格模式支持不好的问题，但后者星星比较多。不过前者没有断开与
后者的关系，但在npm上是两个互相独立的包……（贵圈好乱）

# 示例

## 配置文件

```javascript
/**
 * This is the config file.
 */

'use strict';

/**
 * Modules.
 */

const fs = require('fs');
const extend = require('extend');
const deepFreeze = require('deep-freeze-strict');

const defaultConfig = {
    "port": process.env.NODE_PORT || 4000,
    "mongoDB": {
        "uri": 'mongodb://192.168.3.233/TestDB',
        "options": {
            "server": {
                "socketOptions": {"keepAlive": 1},
                "poolSize": 10
            }
        }
    },
    "cors": {
        "exposeHeaders": ["Accept-Ranges", "Content-Encoding", "Content-Length", "Content-Range", "Authorization"],
        "maxAge": "3600"
    }
};

const configs = {
    "development": {
        "mongoDB": {
            "uri": 'mongodb://127.0.0.1/TestDB'
        }
    },
    "production": {
        "port": 5000,
        "mongoDB": {
            "uri": 'mongodb://thisisusername:thisispasswd@thisishost1:3717,thisishost2:3717/mainDB?replicaSet=thisissetname',
            "options": {
                "auth": {"authdb": "admin"}
            }
        }
    }
};

module.exports = deepFreeze(
    extend(
        true,
        defaultConfig,
        (configs[process.env.NODE_ENV] || configs.development)
    )
);

```

## 在程序入口引入配置文件一次

```javascript
/**
 * Config.
 */
global.CONFIG = require('./config');
```

## 在任意地方调用配置项

```javascript
/**
 * Basic type.
 */
const maxAge = CONFIG.cors.maxAge;

/**
 * Object.
 */
const mongoOptions = Object.assign({}, CONFIG.mongoDB.options);
```
