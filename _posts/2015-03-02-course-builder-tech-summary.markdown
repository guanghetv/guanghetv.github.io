---
layout: post
title:  "课程后台技术总结"
categories: tech
author: "gynsolomon"
---    
    本文是我在洋葱数学“课程后台”项目过程中积累的技术经验介绍和遇到一些问题的解决方案记录。

    本文主要包含以下几个方面：

        1. Material Angular 及 Flex 布局的简介，及在产品中的应用
        2. ui-router 前端路由框架的介绍，以及其与 ngRoute 结合使用的方法
        3. ApiService 的实现方式，以及课程数据结构的provider
        4. 制作自定义组件<sortable-tabset/>的原因及使用方法介绍
        5. 交互视频模块的实现方式
        6. 对课程后台产品未来形态的思考及建议


    ##一、Material Angular 及 Flex 布局 （https://material.angularjs.org）

    Material Angular(以下简称MA) 是一组 Material Design(以下简称MD) 风格的 angular directives & services。
    关于MD，这是在 Google 2014年 I/O 大会上崭新亮相的一种用户界面风格和设计规范，更详细的介绍可以查阅 http://www.google.com/design

    而 Flex 布局，是 w3c 在 CSS3中引用的一种叫做flexbox的技术标准。MA在自己的CSS文件中对其进行了封装，用户只需要在html tag中使用很简单的attribute，就可以完成丰富且响应式的界面布局。
    有过Android开发经验的开发者应该对xml布局中各种layout布局不会陌生吧。只需要定义LinearLayout、RelativeLayout等外围布局结构，再辅以部分属性配置，就可以很快地实现复杂页面的界面框架。
    flex布局的使用方法非常类似，也是基于行列排布来实现DOM的位置和格局控制。相比bootstrap的grid system, flex更加简洁，当然灵活性和定制性方面就相应地差一些。

    课程后台的特殊性在于，它是一款面向洋葱数学课程编辑团队内部的产品，对界面的要求并不像前台产品那样需要高定制化、个性化。简而言之，这款产品最大的价值就是它的实用性。所以在开发过程中，除了前期
    有相关产品人员出具产品需求和页面线框图之外，项目过程中完全没有设计人员参与其中。为了最大程度地保证产品的用户体验和界面美观度，这就需要开发人员除了设计产品逻辑架构外，还要兼起交互设计甚至
    一部分UI设计的工作。在这样的现实情况下，使用MA，大大减轻了这部分的工作量，可以看到课程后台项目中大量使用MA的组件和服务，也保证了界面风格的一致；而flex小巧且简便的布局方式也很好地满足了并不
    苛刻的界面布局要求。

    以$mdDialog这个service为例，只需要将该服务注入到需要使用的地方，调用.alert或者.show方法，即可快速地在页面前端展现一个对话框，后者是用来实现自定义界面的方法，可以像定义directive那样
    传入template、locals本地变量还有resolve一些promise等等。而在html中，使用<md-dialog></md-dialog>这样一对标签，就可以快速地将想要展现的内容封装起来。flex方面，课程后台界面中存在
    大量以左右导航，上下排列实现的界面布局，对于html来说，在<div>标签中添加 layout 属性，即可方便配置，<div layout="row" layout-align="center center" layout-margin></div>
    这样的一条语句即可将放置在div内部的元素按照行排列，且实现上下左右全部居中且智能安排margin的效果，就是这么简单！


    ##二、ui-router 前端路由框架 （http://angular-ui.github.io/ui-router）

    ui-router 是 angular-ui项目组下面的一个子项目。关于angular-ui，更详细的了解可以查阅 http://angular-ui.github.io ，这个可以称为angular前端开发的宝库也不为过。

    在angular单页面应用开发中，配置页面跳转的URL, 以及为每个URL绑定controller, 最常见的做法是使用$routeProvider，这是属于ngRoute这个module的一个provider. 使用.when方法可以很方便
    地配置每个子页面（partial）。而ui-router其实做了大部分相同的工作，区别在于，ui-router引进了一个叫做"state"的概念。ui-router内部不再是子页面的跳转，而是state的跳转，子页面与state
    不必一一对应，它们的链接和嵌套通过<ui-view/>这个标签来索引。在任何页面内部，如果需要发生state跳转，只需要在此页面的相应位置写入<ui-view/>标签，然后调用$state.go('xxxx')方法，就可以
    将子state对应的html模板填充到ui-view的位置上来。

    在课程后台中，因为课程层级层层嵌套的缘故，在编辑过程中，几乎每个层级都需要对子层级的内容进行导航和索引。反映到UI界面上，就是大量地使用了标签页、侧边栏导航等控件。而这恰恰是ui-router的强项。
    关于这一点，可以在这个官方的Demo中看到相关效果（http://angular-ui.github.io/ui-router/sample/#/contacts）。导航栏中的每一个item都对应着一个itemId或者index，通过调用.go方法时
    传参的方式，将itemId传给子state, 就可以让子state绑定的模板显示出对应着该ID的子内容。我猜测，也正因为这种设计使得UI界面跟state还有路由框架紧密联系在一起，所以ui-router才将UI两个字母
    跟router放在了一起，共同组成了这款前端路由框架的名字。

    配置state的时候，如果层级嵌套特别深，就会造成这部分代码长度过长，影响阅读。有一个很方便的第三方library解决了这个问题(https://github.com/marklagendijk/ui-router.stateHelper), 课程
    后台代码中也使用了stateHelper来配置state。

    现在问题来了，如果我不想放弃ngRoute，但又想尝试ui-router新鲜方便的功能怎么办？在这个问题上，课程后台项目开发过程中，走过了一个天大的坑。$routeProvider 和 $stateProvider 配置页面路由，
    都是基于白名单机制的，凡是不在各自的配置中的URL，遇到后，要么报出404 error，要么遵循otherwise路径去导向预先配置的页面。而ngRoute 和 ui-router又都有各自不同的otherwise provider,
    这导致如果在项目中直接同时使用这两个module，它们会互相争夺对前端路由的控制权，最终出现不可预料的结果。在经历了无数的错误尝试和痛苦之后，我们逐渐摸索到了让两者和平共处的方式，并最终形成了这段代码
    https://gist.github.com/gynsolomon/bfee1c81b2eb478d896f 。实现方式是让$stateProvider优先取得路由的控制权，然后在otherwise方法中去匹配$routeProvider中的全部URLs，只要满足当前
    URL属于这些URLs的情况，就将页面路由控制权导向routeProvider, 从而实现两者白名单的融合。

    顺便提一句，需要在index.html <body>中同时填入<ng-view/> 和 <ui-view/>， 因为这是两者各自依赖的顶级入口。


    ##三、前后端 API 和数据通路

    这次项目开发最顺畅的是前后端数据通路部分，因为前后端分离开发的缘故，在我着手写前端应用的时候，后端的数据库设计、课程数据结构以及前后端接口的API已经基本确定。
    在cb/components目录下，有一个services文件夹，这里存放的js文件大部分都是与数据通路有关系的。

    其中 ApiService 中，有一段代码是设置了一个value: API_URL, 然后给它赋值了一个特别大的object。仔细观察，这个object的key/value恰恰就是API的方法名和URL！

    之所以将所有课程相关的API在前端再用这样的形式封装一遍，是因为node后端不仅有课程的API，还有许多其他API，比如用户系统等。这样抽离出来的优点，一是为了集中，方便查阅；二是为了简便，任何地方需要访问
    后端数据时，只需注入Api这个service，然后调用相关方法，并传入参数即可，无需再将繁琐的URL引入其他js文件中。除此之外，customHttp这个service将 angular 原生的$http与$q这两个service进行了封装，
    这样Api这个service的所有方法对外返回的都是标准的$q的promise，而调用时无需再使用deffer, 大大缩减了代码复杂度。

    CourseStructure这个service，就是所谓的课程数据的provider。课程后台前端界面中，有许多关于课程层级的添加按钮。比如添加基础、提高、挑战等任务，再比如添加交互、巩固等模块。这些地方如果写死在html中，
    将来课程结构如果发生变动，调整起来难度就非常大。而CourseStructure这个service就相当于一个配置文件，任何课程层级的结构，都可以在这里进行配置，包括它们的name、type、title等等属性。这样只要在HTML
    需要的地方，ng-repeat一下，就可以将该层级所有的类型都展现出来。这可以算作一个content provider。


    ##四、<sortable-tabset> directive (https://github.com/guanghetv/sortable-tabset)

    这是一个我自己制作的angular directive, 并且将它放在了公司的github账号下面。因为是开源项目，所以其他开发者理论上也可以拿去用的。
    它的主要功能是一个带有排序功能的侧边导航栏。听起来很简单是吗？对，它的原理确实非常简单，就是将ui-bootstrap(http://angular-ui.github.io/bootstrap/)的tabset控件跟ui-sortable控件进行了结合。ui-bootstrap中的tabset是一个非常方便使用的控件，而且内部的设计非常灵活。这个directive实现了bootstrap原生的tab页切换效果，可以方便的切换横向纵向模式，可以动态绑定每个标签页的模板，所以在课程后台中得到了很多的应用，特别是在课程内容层级的侧边导航栏中。

    课程内容的编辑当然需要对资源进行排序。想当然地，我想到了ui-sortable, 这个简单，直接将ui-sortable作为attribute添加到需要排序的控件组的parent层级，并绑定ng-model就马上可以实现了。于是我进行了这样的操作，将ui-sortable添加到了<tabset>标签里。然后到前端查看效果，可出现的情况是，当拖拽排序时，整个导航栏被整体拖动了，而不是预期的子元素被拖动排序。打开dev tool审查元素，发现ui-sortable这个attr标签出现在了外层的<div>标签里，而不是div里面的<ul>里。这当然不可以了，ui-sortable就是通过识别attr来进行排序的。位置为什么会出错呢？查看tabset源码发现，在它的HTML模板中，ul确实是被嵌入在div中的，也就是说，虽然在使用的时候，<tab>是被直接放在<tabset>下面去ng-repeat的，但是它们被解析之后的DOM结构多了一层div。于是我尝试修改tabset源码，直接在模板的正确位置添加了ui-sortable属性，果然就可以正确排序了。

    这个自制的directive除了做了上述的事情之外，还将需要排序的数据model作为scope参数供给了外部。这样基本上可以满足多种多样的数据排序需求了。

    欢迎fork项目并进行进一步加工。


    ##五、交互视频模块的实现方式
    
    交互视频模块是整个课程后台项目中难度最大也是花费时间最长且需求变化最多的地方。体现在代码上，就是我写了两个版本。在components/directives下可以看到两个名字非常像的文件夹，分别是hyper-point-editor 和 interactive-point-editor。最终在项目中使用的是前者。

    抛开差异，这两个版本的设计思路大致相同。我们的交互视频的交互是体现在选项的schema上的。通过为选项配置跳转条件来实现当用户点选某个选项之后从而跳转到相应视频的功能。而目前课程部门的需求仅要求正确和错误两种跳转，但其实不排除未来有可能会逐选项跳转。为了这种灵活性做考虑，课程后台在实现方式上做了特殊处理。代码中，hyper-point-editor这个directive被调用时，不是只被调用了一次，而是采用的ng-repeat的方式，repeat的母体则是一堆choices.这堆choices就是该交互题的所有选项按照正确和错误分类之后的选项集合。但是UI界面上却只要求正确或错误跳转逻辑的配置只有一个地方，于是又在代码中添加了ng-show来判断repeat的$index,只显示第一个。这样当课程编辑在编辑答对或者答错的跳转逻辑时，其实只是配置了第一个正确选项和第一个错误选项的跳转逻辑，当点击保存时，课程后台会将这部分逻辑赋值给其他正确选项（这个目前没有意义，因为正确答案目前只有一个）和错误选项，然后向数据库发起保存请求，从而实现所有的正确选项或者错误选项的跳转逻辑保持一致。

    那么这两个版本的差异，主要体现在界面和默认值上面。因为上述的设计，使得配置跳转逻辑相当灵活，interactive-point-editor这个版本就索性将所有的域开放给课程编辑，甚至不怎么区分是跳转到主视频还是分支视频，全凭课程编辑录入的url来判断。而且当时的schema也支持这种形式。当后来随着洋葱数学前台关于这部分逻辑的调整（有一封邮件），以及课程编辑感觉全部放开的权限反而使其录入工作变得繁琐，于是更新了第二个版本，就是hyper-point-editor. 这个版本在界面上做了简化，默认跳转到主视频并赋值给jumpVideo.time = null.

    这个directive确实体积庞大，其中不仅自身是封装，还用到了time-transformer 和 video-selector这两个directive。


    ##六、对课程后台产品未来形态的思考及建议 

    这里简单写一下提纲，更多的内容需要更加仔细的思考和Demo论证：

    1. 从教材到章节到知识点…一直到子任务层级，均属于课程结构构建过程，用json比用UI界面简单不知多少倍；
    2. 习题和视频是真正的资源和学习内容，且构建内容相对复杂，可以用UI界面来制作；
    3. 全UI界面的形式，一旦将来课程结构发生变化（比如知识点层级没了），对现有代码库来说，将是一场灾难；
    4. 更多参考 Khan Academy (http://khan.github.io/perseus/) 这个项目，在真正的学习内容构建上多花功夫。
