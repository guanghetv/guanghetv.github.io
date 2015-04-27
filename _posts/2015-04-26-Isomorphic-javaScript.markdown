## Isomorphic JavaScript
随着Javascript不断的发展，前端出现很多基于单页面的MVC框架如:Anagular.js,Backbone.js,Ember.js等，使得开发人员非常容易的开发富Javascript应用，但是这些技术也有局限性，下面我们将从单页面说起，再说它们的局限性，以及解决办法。

### The Single-Page App

在单页面中许多应用逻辑（视图，模板，控制器，模型，国际化）在前端完成，前端JS通过服务端（服务端可以使用任何语言）API获得数据，并建立数据建模，通过业务控制器处理业务逻辑，通过UI控制器渲染HTML页面。

![alt MVC](http://nerds.airbnb.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-06-at-5.21.00-PM.png =400x400)

单页面的优势是：

  1. 对于用户而言，一旦前端完成JS初始化并加载数据，用户可以很快速的浏览页面而不需要刷新页；
  2. 对于开发者而言，单页面开发分离了前后端的关注点，避免了前后端过多的逻辑共享;
  
但单页面有局限.

### Trouble in Paradise
	
  1. SEO  	
   一个只能运行在前端的JS应用不能为搜索引擎爬虫提供HTML，因为网页爬虫通过分析请求网站返回的HTML来获取SEO相关信息;
  
  2. Performance
  
	因为单页面应用在用户看到内容之前会花很多时间去初始化（加载JS脚本，从服务端获取数据），在这段时间用户会看到空白的页面或者加载动画，这回影响用户体验，造成用户流失。而且因为JS是单线程，过多的逻辑放到前端，会造成HTML页面渲染和动画效果不流畅;
  	
  3. Maintainability
  
  	有些逻辑比如：时间，日期，货币的格式化，验证还是会在前后端共享，所以在修改这些逻辑的时候会同时修改前后端。增加了维护的难度；
  	
### A Hybrid Approach

Javascript应用同时运行在前后端，能够解决上面的（SEO,Performance,Maintainability）问题

![alt Hybrid Approach](http://nerds.airbnb.com/wp-content/uploads/2013/11/Screen-Shot-2013-11-07-at-10.29.32-AM.png =400x400)

### Isomorphic JavaScript in the real world

1. [Meteor.js](https://www.meteor.com/)
2. [Browserify](http://browserify.org/)
3. [Angularjs-server](https://github.com/saymedia/angularjs-server)

### Original text

[Isomorphic JavaScript: The Future of Web Apps](http://nerds.airbnb.com/isomorphic-javascript-future-web-apps/)

