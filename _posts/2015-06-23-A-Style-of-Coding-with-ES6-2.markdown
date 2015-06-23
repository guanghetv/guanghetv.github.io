---
layout: post
title:    "A Style of Coding with ES6 2"
category: tech
author: "libook"
---

**这期主要讨论利用let、const和块级作用域来更合理地使用作用域。**

在ES6出来之前，Javascript中只能使用var命令来声明变量，甚至在不使用严格模式的情况下，可以不使用var命令声明来直接使用变量；
这样固然方便，但是随着软件规模的增大以及开发人员的增多，由作用域不严谨导致的问题接踵而至：

- 多处的变量名重复，导致一处的变量值被莫名其妙的覆盖了；
- 由于var命令声明的变量存在提升（hoist）的特性，会出现代码执行顺序变化的问题；
- 作用域界线模糊，一个作用域下除了能访问自己所需的变量以外还能访问到大量本不应该访问到的变量，尤其是大量使用全局变量，严重影响调试和执行的效率，也使得程序不稳定；
- 通常使用立即执行匿名函数（IIFE）来达到一定的隔离作用域的作用，但是需要写很多代码，且效果并不好；
- 由于之前不存在常量类型，用var定义的逻辑上的常量，实际上也会被修改；而且如果没有特定的命名规则或注释说明，编程者也不会意识到不应该修改它。

那么，现在ES6来了，为我们带来了let、const和块级作用域来更好地规划作用域。

##let命令##

ES6新增了let命令，用来声明变量；
它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效；
let不可重复声明；
let不存在变量提升的特性。

##const命令##

const用来声明常量；
一旦声明，常量的值就不能改变；
除此之外具有let的所有性质。

不过有一点需要注意，使用const命令声明的对象是可以变化的。
因为const命令只是指向变量所在的地址，这个地址指向一个对象；
不可变的只是这个地址，即不能把此变量指向另一个地址，但对象本身是可变的，所以依然可以为其添加新属性。
如果真的想将对象冻结，应该使用Object.freeze方法：

```javascript
//将对象本身冻结，但属性内部不冻结
const foo = Object.freeze({});
foo.prop = 123; // 不起作用

//以下是将对象彻底冻结
var constantize = (obj) => {
    Object.freeze(obj);
    Object.keys(obj).forEach( (key, value) => {
        if ( typeof obj[key] === 'object' ) {
            constantize( obj[key] );
         }
    });
};
```

##块级作用域##

使用大括号“{}”括起来的代码被称为一个“块（block）”，我们之前看到过function块、if块、for块等等，
块可以将作用域分割成内外两部分，之所以使用立即执行匿名函数能够达到作用域隔离的作用也是因为使用使用了function块。
在ES6中提供了一个简单的方式来构建一个块，就是直接使用大括号将代码括起来：

```javascript
// 块级作用域写法
{
    let tmp = ...;
}
```

块级作用域会出现ES5与ES6的兼容问题：

```javascript
function f() { console.log('I am outside!'); }
(function () {
    if(false) {
         // 重复声明一次函数f
        function f() { console.log('I am inside!'); }
    }

    f();
}());
```

上面代码在ES5中运行，会得到“I am inside!”，但是在ES6中运行，会得到“I am outside!”；
这是因为ES5存在函数提升，不管会不会进入if代码块，函数声明都会提升到当前作用域的顶部，得到执行；
而ES6支持块级作用域，不管会不会进入if代码块，其内部声明的函数皆不会影响到作用域的外部。

需要注意的是，如果在严格模式下，函数只能在顶层作用域和函数内声明，其他情况（比如if代码块、循环代码块）的声明都会报错。

##关于全局变量##

ES6规定，var命令和function命令声明的全局变量，属于全局对象的属性；
let命令、const命令、class命令声明的全局变量，不属于全局对象的属性。

##ES6的类没卵用##

虽然ES6推出了类的概念，使其看似更像是真正的面向对象的语言；
虽然有了继承和构造函数等显得更加“规范”，但是由于其暂时不具有封装的特性，
所以并不能像真正的面向对象的语言那样用，使用类实现出的对象，其所有成员变量和方法均可以在外部访问，甚至修改；
所以为了程序的稳定性，暂且不建议使用类；
建议还是使用ES5时期的构造函数的方式来生成对象，该暴露的暴露，该隐藏的隐藏；
除非有什么其他的更科学的方法来使用类。

---

#建议代码风格#

1. 除非特殊情况的需要，全局使用let来声明变量，常量一定使用const声明；
2. 充分利用块级作用域，尽可能让一个块内只能访问到其必要访问的变量；

风格示例：

```javascript
/**
 * Caesar Cipher
 * @param {Object} options - An object contain offset, keyWords.
 * @constructor
 */
const Caesar = function (options) {
    let offset = (options.offset === undefined) ? 0 : options.offset;
    let keyWords = (options.keyWords === undefined) ? '' : options.keyWords.toUpperCase();

    {
        let keyWordsCopy = keyWords;
        keyWords = '';
        for (let kwci in keyWordsCopy) {
            let copyWord = keyWordsCopy[kwci];
            if (/\s/.test(copyWord)) {
                continue;
            }
            let found = false;
            for (let kwi in keyWords) {
                if (keyWords[kwi] === copyWord) {
                    found = true;
                    break;
                }
            }
            if (!found) {
                keyWords += copyWord;
            }
        }
    }

    let newAlphabet = alphabet;
    {
        let fragment = newAlphabet.substring(0, offset);
        newAlphabet = newAlphabet.substring(offset) + fragment;
    }
    {
        for (let kwi in keyWords) {
            let keyWord = keyWords[kwi];
            let regexp = new RegExp(keyWord, 'igm');
            newAlphabet = newAlphabet.replace(regexp, '');
        }
        newAlphabet = keyWords.concat(newAlphabet);
    }

    let comparisonTable = {};
    {
        for (let ai in alphabet) {
            let letter = alphabet[ai];
            comparisonTable[letter] = newAlphabet[ai];
        }
    }

    let substitution = new Substitution({"comparisonTable": comparisonTable});
    this.encrypt = substitution.encrypt;
    this.decrypt = substitution.decrypt;
};
```

