---
layout: post
title:  "Appium+nodeJS测试用例编写"
category: tech
author: "lucien"
---


#一：安装mocha测试框架
###1.命令行安装
######进入到当前目录下,命令行安装:

```
npm install mocha 
```

#二：学习官网的案例
###1、各种语言的脚本案例:
<https://github.com/appium/sample-code>

######2、我用的是Node建议吧node文件夹下载下来:
<https://github.com/appium/sample-code/tree/master/sample-code/examples/node>

#三:自己写nodeJS脚本
###1. 结合案例脚本与Appium生成的脚本,自己开始独立写脚本.
######下面是自己写的从登录-观看视频-滑动视频的脚本:

```
"use strict";
var          wd = require("wd"),
           chai = require("chai"),
 chaiAsPromised = require("chai-as-promised"),
        actions = require ("./helpers/actions");

var      driver = wd.promiseChainRemote("0.0.0.0",4723),
        desired = {
            "Appium-version":"1.4.13",
            platformName:"iOS",
            platformVersion:"9.3",
            deviceName:"iPhone 6",
            app:"/Users/lucien/Library/Developer/Xcode/DerivedData/YCMath345-iOS-crxbfrnkjtohbydohibkqgwayzos/Build/Products/Debug-iphonesimulator/YCMath345-iOS.app"
                  };
wd.addPromiseChainMethod("swipe",actions.swipe);

// before(function () { return driver.init(desired) });
//  after(function () { return driver.quit() });




describe("一:滑动视频啊", function () {
    this.timeout(300000);
  
    before(function () { return driver.init(desired) });

    it("1.1:展示滑动视频案例.", function () {
        return  driver
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[3]").click()
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIATextField[1]").sendKeys("15725040100")
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIASecureTextField[1]").sendKeys("123456")
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[3]").click()    //点击登录
            .sleep(3000)
           // .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[8]").click()    //点击我知道了
          //  .sleep(3000)
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[3]").click()    //点击蒙版
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIATableView[1]/UIATableCell[2]/UIAButton[1]").click()   //点击相交线
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIATableView[1]/UIATableCell[2]").click()          //点击什么是对顶角
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[13]").click()        //点击跳过(片头)
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[1]").click()       //点击蒙版
            .elementByXPath("//UIAApplication[1]").click()       //点击屏幕
            .sleep(3000)
           // .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIASlider[1]").getLocation()

            .elementByXPath("//UIAApplication[1]/UIAWindow[1]").getLocation()     //获取当前windows跟获取当前slider都行.
            // .then(function(loc){
            //     return driver.swipe({startX:loc.x,startY:loc.y,endX:loc.x,endY:500,duration:800})
            // })
             .then(function(loc){
                return driver.swipe({startX:50,startY:50,endX:50,endY:650,duration:800})
            })
            .sleep(3000)
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[12]").click()             //点击跳过(片尾)
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIAButton[2]").click()       //点击休息一下.
            .elementByXPath("//UIAApplication[1]/UIAWindow[1]/UIATableView[1]/UIATableCell[1]").click()  //点击当前cell,验证一下是否返回到这这个界面
            .quit()
    });

});
  
```

#四:疑问?
###1.提示找不到"XXX.module"
```
npm install xxx
```
或者

```
npm install -g xxx
```
###2.简述下mocha模块怎么用?
#####除去导入以及定义,基本分3个模块.
######1.before:
即在每个用例执行前需要走的代码.这里是初始化运行模拟器,并将其与本地appium服务器连接.
######2.after:
即在每个用例执行后需要走的代码,一般是关闭模拟器.
######3.decribe:
描述,就是给用例加上一个类似一级标题的描述.在这个函数中利用it函数,前面就是子标题描述,后面是函数方法.
this.timeout(300000);设置超时5分钟,必须加上.
describe可以嵌套.
###3.库.模块怎么引入?
######库:require("xxx");
######模块:require("./helpers/actions");
(这是项目中用到的一个手势动作,利用了本地库的actions)
###4.滑动是滑动哪个组件?
滑动当前的UIAWindow或者UIASlider都可以.先用getLocation()获取组件的位置,然后swipe滑动,此函数里面有五个属性,startX:开始的x值 startY:开始的y值 endX:结束的x值 endY:结束的y值 duration:滑动时间(可以理解为延迟时间),完成手势操作.
###5.如何运行nodeJS的用例?
```
mocha  xxx.js
```
###6.其他关于测试用例的问题?
可自己研究Apppium的案例代码,里面几乎包含了所有操作.