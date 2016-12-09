---
layout: post  
title:  "如何使用react native中的Navigator组件"  
category: tech  
author: "yinxin630"
---

## Navigator用法

Navigator示例代码  

```
import React from 'react';
import { Navigator } from 'react-native';
import page1 from './page1';

export default class SampleNavigator extends React.Component {
    render() {
        return (
            <Navigator
                // 默认页面, name: 默认页面组件名, component: 默认页面渲染组件
                initialRoute={{ name: 'page1', component: page1 }}
                // 页面切换动画
                configureScene={(route) => {
                    return Navigator.SceneConfigs.VerticalDownSwipeJump;
                }}
                // 将页面参数和navigator注入渲染组件中
                renderScene={(route, navigator) => {
                    let Component = route.component;
                    return <Component {...route.params} navigator={navigator} />
                }} 
            />
        );
    }
}
```

`page1.js`代码:  

```
import React, { Component } from 'react';
import { StyleSheet, Text, View } from 'react-native';
import page2 from './page2';

class Page1 extends Component {
    componentDidMount() {
        setTimeout(() => {
          this.props.navigator.push({
              name: 'page2',
              component: page2,
          });
      }, 2000);
    }
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.welcome}>
                    Welcome to Page 1!
                </Text>
            </View>
        );
    }
}

const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    welcome: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
});

export default Page1;
```

## Navigator路由跳转

要被渲染的`page1`组件的props中会包含一个`navigator`属性, 该属性用于页面跳转, 如`componentDidMount`中通过`push`函数使页面跳转到`page2`.

navigator包含一个页面栈, 通过操作这个栈来实现页面跳转, navigator包含的方法有:

* getCurrentRoutes() - 获取当前栈里的路由，也就是push进来，没有pop掉的那些
* jumpBack() - 跳回之前的路由，保留现在的路由，还可以再跳回来
* jumpForward() - 上一个方法不是调到之前的路由了么，用这个跳回来
* jumpTo(route) - 跳到之前的某个路由, 不删除现有路由
* push(route) - 向栈中添加一个新路由
* pop() - 向栈中删除一个路由
* replace(route) - 用一个新路由替代当前路由
* replaceAtIndex(route, index) - 用一个新路由替换指定路由
* replacePrevious(route) - 用一个新路由替换上一个路由
* immediatelyResetRouteStack(routeStack) - 使用一个路由数组替换栈中路由
* popToRoute(route) - 跳到指定路由并清除栈中路由
* popToTop() - 跳到栈顶路由并清除栈中路由

## 向渲染组件传递参数

在跳转路由时, 可以通过额外的`params`参数来向组件传递props.

示例: 

```
this.props.navigator.push({
    name: 'page2',
    component: page2,
    params: {
        hello: 'hello',
        other: 'other props',
    },
});
```

## 支持的跳转动画类型

跳转动画的定义位于: `node_modules/react-native/Libraries/CustomComponents/Navigator/NavigatorSceneConfigs.js`  

当前(`v0.30.0`)支持的动画类型: 

* PushFromRight
* FloatFromRight
* FloatFromLeft
* FloatFromBottom
* FloatFromBottomAndroid
* FadeAndroid
* HorizontalSwipeJump
* HorizontalSwipeJumpFromRight
* VerticalUpSwipeJump
* VerticalDownSwipeJump