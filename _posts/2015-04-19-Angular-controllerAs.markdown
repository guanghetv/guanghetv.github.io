---
layout: post
title:  "Angular controllerAs and $scope"
category: tech
author: "YshadowZ"
---
## 先来看一个比较常见的问题

```javascript
var myApp = angular.module('myApp',[]);
myApp.controller('parentController', function($scope){
	$scope.name = 'xiaoming';
});
myApp.controller('sonController', function($scope){
	/*no code*/
});
```

```
<div ng-app="myApp">
	<div ng-controller="parentController">
		<input ng-model="name">
		<div ng-controller="sonController">
			<input ng-model="name">
		</div>
	</div>
</div>
```

****

这样，当我们在``parentController``管辖下的``<input>``输入框中输入内容时，``sonController``下的``name``变量也会随之更新，但是当我们更新``sonController``中的``name``变量时，``parentController``中的``name``变量并不会随着一起更新。

因为javascript是基于原型链的继承。``parentController``与``sonController``中的``$scope``是父子关系，当``sonController``中的``$scope``想要取``name``的值的时候，没有的情况下，会沿着原型链往上寻找，即看看``parentController``下的``$scope``中是否有此值，以此类推，所以当我们更新``parentController $scope.name``时，由于``sonController``管理下的页面也需要渲染这个值，但是它自己没有，所以将``parentController scope``下的``name``值输出了出来。

反过来，当我们更新``sonController``中的``name``值的时候，由于存在属性遮蔽，set操作只会发生在sonController的scope下面，不会更新``parentController scope中的``name``的值。所以，我们在编程过程中涉及到作用域嵌套，并且在父子``controller``中都操作``$scope``下的同名基本数据类型变量往往会遇到这个问题。

传统的解决办法是将这些想要共用的基本数据类型都挂在一个对象下面，像这样：

```javascript
var myApp = angular.module('myApp',[]);
myApp.controller('parentController', function(){
	$scope.peopleObj = {};
	$scope.peopleObj.name = 'xiaoming';
});
myApp.controller('sonController', function(){
	/*no code*/
});
```

```
<div ng-app="myApp">
	<div ng-controller="parentController">
		<input ng-model="peopleObj.name">
		<div ng-controller="sonController">
			<input ng-model="peopleObj.name">
		</div>
	</div>
</div>
```

更加优雅的解决办法是使用controllerAs属性。如下：

```javascript
var myApp = angular.module('myApp',[]);
myApp.controller('parentController', function($scope){
	this.name = 'xiaomming'
});
myApp.controller('sonController', function(){
	/*no code*/
});
```

```
<div ng-app="myApp">
	<div ng-controller="parentController as parCtl">
		<input ng-model="parCtl.name">
		<div ng-controller="sonController as sonCtl">
			<input id="son" ng-model="sonCtl.name">
			<input id="parent" ng-model="parCtl.name">
		</div>
	</div>
</div>
```

再次运行程序，我们可以看到，当改变``parentController``下的``name``值时，``<input id="parent">``中的值会跟随变化，而``<input id="son">``中的值不会变化，改变``<input id="son">``中的值，``parentController``下的``name``值不会变化，但改变``<input id="parent">``中的值，``parentController``中``name``的值会随之改变。而且很清楚的知道用的是哪个``scope``下的变量。

## controllerAs的实质

虽然上面的例子中，变量都挂在了``this``下面，但是也和页面实现了双向绑定，因此，实际的实现机制中，``this``还是关联在了``$scope``下面。以下这两种实现方式并无差异：

* 1.

```javascript
myApp.controller('parentController', function(){
	this.name = 'xiaoming';	
});
```

```
<div ng-controller="parentController as parCtl">
	<input ng-model="parCtl.name">
</div>
```

****

* 2.

```javascript
myApp.controller('parentController', function($scope){
	$scope.parCtl = this;
	this.name = 'xiaoming';	
});
```

```
<div ng-controller="parentController">
	<input ng-model="parCtl.name">
</div>
```

## 在使用controllerAs中遇到的问题
如下：

* view:

```
<div ng-app="myApp" ng-controller="parentController">
	<isolate me="me"></isolate>
    <button ng-click="change()">change</button>
</div>
```

* controller

```javascript
var myApp = angular,module("myApp",[]);
myApp.controller('parentController', function($scope){
	$scope.me = {name: 'xiaoming', age: 12};
	var newPeople = {name: 'xiaohong', age: 14};
	$scope.change = function(){
        $scope.me = newPeople;
    };
});
```

* directive

```javascript
myApp.directive('isolate', function(){
	return{
		restrict: "E",
		scope: {
			me: "="
		},
		templateUrl: 'isolate.html',
		controller: function($scope){
			this.me = $scope.me;
		};
		controllerAs: "isolateCtl"
	}
});
```

* directive view

```
<div>
	<div>{{isolateCtl.me.name}}</div>
	<div>{{isolateCtl.me.age}}</div>
</div>
```

这样，当单击页面上change按钮时，isolate模板中的内容并没有同步变化，但是``$scope.me``是确确实实的更新了。如果我们在实现isolate directive时，没有使用``controllerAs``,那么这个变化确实是同步的。

** 问题的原因： **
``parentController``中，``$scope.me``最初绑定的是``{name:"xiaoming",age:14}``的引用,单击change按钮后，被赋值成了newPeople的引用，引用发生了变化，会触发``$digest``循环，同步更新view，但是在使用了controllerAs的directive中，使用了``this.me=$scope.me``，也就是说``$scope.me``中的值又在directive 的``$scope.isolateCtl.me``下面存了一份，虽然$scope.me的值更新了，指向了新的引用，但是``$scope.isolateCtl.me``的值并没有更新，依旧指向``$scope.me``原来指向的Object，所以导致了directive模板中的内容没有被更新这种现象。

## 资料参考
http://codetunnel.io/angularjs-controller-as-or-scope/