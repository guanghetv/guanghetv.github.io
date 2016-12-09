---
layout: post
title:  "Angular 代码优化"
category: tech
author: "YshadowZ"
---
一些老生常谈的注意事项比如文件组织（个人认为这个应该是见仁见智的，只要一个团体形成一个统一的规范就好），单一职责；
一些基础知识，比如数据绑定，多重视图和路由，控制器，基本指令，过滤器，服务，依赖注入，模块就不在这里唠叨了。关于基础知识的夯实和扩展请看[这里](http://www.ng-newsletter.com/advent2013/#!/)。
通过一段时间的使用以及对一些文章的阅读，总结了一些在使用Angular编程时的优化点，有不妥的地方，欢迎大家指出。

***

###在使用$routeProvider配置路由时使用controllerAs属性
通过使用controllerAs属性能够更加清晰的看出试图中绑定的变量所属的控制器。例子如下：

```javascript
angular.module('Test', ['ngRoute', 'Test.controllers'])
    .config(function($routeProvider){
        $routeProvider.when('/', {
            templateUrl: './test.html',
            controller: 'TestController',
            controllerAs: 'Test'
        })
    })
```

```javascript
angular.module(Test.controllers, [])
    .controller(TestController, function(){
        var Test = this;
        Test.showMessage = function(){
            console.log(Test.peopleName);
        }
    })
    .controller(InnerTestController, function(){
        var InnerTest = this;
        InnerTest.showMessage = function(){
            console.log(InnerTest.peopleName);
        }
    })
```

```html
<input ng-model='Test.peopleName' />
<button ng-click='Test.showMessage()'>showPeopleName</button>
<div ng-controller='InnerTestController as InnerTest'>
    <input ng-model='InnerTest.peopleName' />
    <button ng-click='InnerTest.showMessage()'>showInnerPeopleName</button>
</div>
```  

**在这里，$scope的$on,$broadcast,$emit并不适用于Test或InnerTest，但并不影响在controller中继续使用$scope作为$on,$broadcast,$emit的载体，controllerAs属性只是使的视图上绑定的变量更加清晰明了。**

***

###优化$digest()调用
在改变一个变量时，通常能够确定在什么情况下来运行$digest循环，以及运行$digest循环会影响哪些作用域。在这种确定的情况下，我们无需运行$scope.$apply()，因为这样的话，相当于在
$rootScope上运行$digest循环，这样很可能会导致性能低下，甚至发生关于$rootScope $digest的错误（曾经遇到过）。在这种情况下，作为替代可以直接调用
$scope.$digest().还用上面的代码举例子:

```javascript
....
.controller(InnerTestController, function($scope){
    var InnerTest = this;
    InnerTest.showMessage = function(){
        setTimeout(function(){
            InnerTest.peopleName = '小明';
            $scope.$parent.Test.peopleName = '小红';
        },2000);
    }
})
```

这样写，在单击showInnerPeopleName按钮2s后,input中的值不会被更新。如果使用$scope.$apply(),如下：

```javascript
setTimeout(function(){
    $scope.$apply(function(){
        InnerTest.peopleName = '小明';
        $scope.$parent.Test.peopleName = '小红';
    });
},2000);
```

则父子input中的值都会更新。若使用$scope.$digest(),如下:

```javascript
setTimeout(function(){
    InnerTest.peopleName = '小明';
    $scope.$parent.Test.peopleName = '小红';
    $scope.$digest();
},2000);
```

则只会更新子input中的值。
**调用$scope.$digest()只会在调用了$digest()及其子节点的具体作用域上运行$digest循环。**

***

###指令类型与指令作用域

- 标签类型的指令一般拥有分离的作用域，如下

    ```javascript
    angular.module('Test', [])
        .directive('testDirective', function(){
            return {
                restrict: 'E',
                scope: {}
            }
        })
    ```

- 属性类型的指令一般拥有继承的作用域，如下

    ```javascript
    angular.module('Test', [])
        .directive('testDirective', function(){
            return {
                restrict: 'A',
                scope: true
            }
        })
    ```

**(个人理解:标签用来展示数据，属性用来修饰数据，对于展示，即不与其他标签的数据相互影响，则作用域是独立的。对于属性，用来修改标签所展示的数据，因此通过继承来得到标签作用域下的数据)**

***

###进入下一个视图前准备好要使用的数据
配置路由时，使用resolve属性，在这里面初始化好一些需要在controller中使用的数据，如下

```javascript
$routeProvider.when('/', {
    template: '',
    controller: 'TestController',
    resolve: {
        userData: function(DataProvider){
            return DateProvider.getUser();
        }
    }
})
```

DataProvider是一个服务，DataProvider.getUser()返回的是一个promise,resolve会等待每一个promise都有结果后才会继续渲染视图。视图所对应的控制器中择可以直接使用resolve中的数据，比如userData。

***

### 对于初始化后就不会再变化的变量，比如常量，使用一次性绑定，这样减少$watch中的数据。
使用[bindonce](https://github.com/pasvaz/bindonce)这个插件可以很容易的实现。对比举例如下(emails是常量数组)

- 使用ng
{% raw %}
    ```html
    <ul>
        <li ng-repeat='email in emails'>
            <a ng-href='#/from/{{email.sender}}'>{{email.sender}}</a>
            <a ng-href='#/email/{{email.id}}'>{{email.subject}}</a>
        </li>
    </ul>
    ```
{% endraw %}
如果有emails的长度为100,那么这一段代码将产生500个$watch,会很明显的降低应用程序的性能。我们并不需要这么多的永久监控器，绑定一次就够了。

- 使用bindonce

{% raw %}
    ```html
    <ul>
         <li bindonce='email' ng-repeat='email in emails'>
             <a bo-href-i='#/from/{{email.sender}}' bo-text="email.sender"></a>
             <a bo-href-i='#/email/{{email.id}}' bo-text="email.subject"></a>
         </li>
     </ul>
    ```
{% endraw %}

***

###资料参考

- [angularjs-style-guide](https://github.com/gocardless/angularjs-style-guide#parts-of-angular)
- [top-10-mistakes-angularjs-developers-make](http://www.airpair.com/angularjs/posts/top-10-mistakes-angularjs-developers-make)
