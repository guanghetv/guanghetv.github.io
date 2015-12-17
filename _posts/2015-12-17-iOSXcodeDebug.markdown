---
layout: post
title:  "Xcode Debug技巧"
category: tech
author: "jichengsun"
---
  * [开发过程中](#开发过程中)
    * [基本调试工具介绍](#基本调试工具介绍)
    * [BreakPoint](#BreakPoint)
      * [Enable NSZombie Objects（开启僵尸对象）](#Enable NSZombie Objects（开启僵尸对象）)
      * [对于所有异常添加Global BreakPoint（全局断点)](#对于所有异常添加Global BreakPoint（全局断点))
      * [Condational Breakpoints（条件断点）](#Condational Breakpoints（条件断点）)
      * [unrecognized selector send to instancd 快速定位](#unrecognized selector send to instancd 快速定位)
      * [Address Sanitizer](#Address Sanitizer)
    * [UI Debug](#UIDebug)
    * [LLDB调试基本命令](#LLDB调试基本命令)
      * [p和po](#p和po)
      * [expr](#expr)
      * [call](#call)
      * [bt](#bt)
      * [image](#image)
      * [BreakPoint](#BreakPoint)
  * [应用发布后崩溃收集](#应用发布后崩溃收集)
    * [Xcode自带线上应用崩溃日志](#Xcode自带线上应用崩溃日志)
    * [根据crash崩溃报告的内存地址定位到代码位置](#根据crash崩溃报告的内存地址定位到代码位置)
    * [Umeng Debug Tool](#Umeng Debug Tool)
    * [Apple一些关于Debug的官方文档](#Apple一些关于Debug的官方文档)




##开发过程中
###基本调试工具介绍

[Debugging Tools](https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/debugging_with_xcode/chapters/debugging_tools.html#//apple_ref/doc/uid/TP40015022-CH8-SW1)

###UIDebug

[Debugging with Xcode](https://developer.apple.com/library/ios/documentation/DeveloperTools/Conceptual/debugging_with_xcode/chapters/special_debugging_workflows.html#//apple_ref/doc/uid/TP40015022-CH9-SW1)

###BreakPoint

####Enable NSZombie Objects（开启僵尸对象）
Enable NSZombie Objects可能是整个Xcode开发环境中最有用的调试技巧。这个技巧非常非常容易追踪到重复释放的问题。该技巧会以非常简洁的方式打印指出重复释放的类和该类的内存地址。
 
开启方式:首先打开**Edit Scheme**，然后选择**Diagnostics**选项卡，勾选**Enable NSZombie Objects**选项。

####对于所有异常添加Global BreakPoint（全局断点)

工程切换到异常浏览窗口(断电管理视图)，点击下方左侧的**Add Breakpoint**按钮，然后选择**Add Exception Breakpoint**确保可以捕获所有异常。

####Condational Breakpoints（条件断点）

添加一个普通断点，然后右键点击断点选择**Edit Breakpoint**，这时就打开了一个断点编辑器，你可以在这里设置断点条件（以及一些其他的断点设置），然后选择**Done**

####unrecognized selector send to instancd 快速定位
在Debug菜单中Breakpoints->Create Symbolic Breakpoint

在Symbolic中填写如下方法签名

```
-[NSObject(NSObject) doesNotRecognizeSelector:] 
```

####Address Sanitizer
**EXC_BAD_ACCESS**一直是很多开发者的噩梦，因为这个错误很不直观，出现后往往要花很长时间才能定位到错误。苹果这次带来了革命性的提升。

在项目的**Scheme**中**Diagnostics**下，选中**enable address sanitizer**


###LLDB调试基本命令
[LLDB命令大全](http://lldb.llvm.org/lldb-gdb.html)

####p和po
**在调试器中最常用到的命令是p（用于输出基本类型）或者po（用于输出 Objective-C 对象）。**

如下，你可以通过输入po 和 view 来输出 view 的信息:

```
po [self view]
```
随后调试器会输出这个 object 的 description。在这个例子中可能是这样的信息：

```
(UIView *) $1 = 0x0824c800 <UITableView: 0x824c800; frame = (0 20; 768 1004); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x74c3010>; layer = <CALayer: 0x74c2710>; contentOffset: {0, 0}>
```

你可能需要的是 view 下 subview 的数量。由于 subview 的数量是一个 int 类型的值，所以我们使用命令p：

```
p (int)[[[self view] subviews] count]
```
最后你看到的输出会是：

```
(int) $2 = 2
```

细心的朋友可能会发现输出的信息中带有$1、$2的字样。实际上，我们每次查询的结果会保存在一些持续变量中```($[0-9]+)```，这样你可以在后面的查询中直接使用这些值。比如现在我接下来要重新取回$1的值：

```
(lldb) po $1
(UIView *) $1 = 0x0824c800 <UITableView: 0x824c800; frame = (0 20; 768 1004); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x74c3010>; layer = <CALayer: 0x74c2710>; contentOffset: {0, 0}>
```
可以看到，我们依然可以取到之前[self view]的值。

####expr
**可以在调试时动态执行指定表达式，并将结果打印出来。常用于在调试过程中修改变量的值。**


```
expr a=2
```
你会看到如下的输出：

```
(int) $0 = 2
```

很明显可以看出，变量a的值被改变。 除此之外，还可以使用这个命令新声明一个变量对象，如：

```
expr int $b=2
p $b
```
下面的命令用于输出新声明对象的值。（注意，对象名前要加$）

####call
**call即是调用的意思。其实上述的po和p也有调用的功能。因此一般只在不需要显示输出，或是方法无返回值时使用call。**

```
call [self.view setBackgroundColor:[UIColor redColor]]
```
继续运行程序，看看view的背景颜色是不是变成红色的了！在调试的时候灵活运用call命令可以起到事半功倍的作用。

####bt
**打印调用堆栈，加all可打印所有thread的堆栈。不详细举例说明，感兴趣的朋友可以自己试试。**

####image
**image 命令可用于寻址，有多个组合命令。比较实用的用法是用于寻找栈地址对应的代码位置。**
如下代码：

```
NSArray *arr=[[NSArray alloc] initWithObjects:@"1",@"2", nil];
NSLog(@"%@",arr[2]);
```
程序运行这段代码后会抛出下面的异常：

```
*** Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndex:]: index 2 beyond bounds [0 .. 1]'
*** First throw call stack:
(
  0   CoreFoundation                      0x0000000101951495 __exceptionPreprocess + 165
  1   libobjc.A.dylib                     0x00000001016b099e objc_exception_throw + 43
  2   CoreFoundation                      0x0000000101909e3f -[__NSArrayI objectAtIndex:] + 175
  3   ControlStyleDemo                    0x0000000100004af8 -[RootViewController viewDidLoad] + 312
  4   UIKit                               0x000000010035359e -[UIViewController loadViewIfRequired] + 562
  5   UIKit                               0x0000000100353777 -[UIViewController view] + 29
  6   UIKit                               0x000000010029396b -[UIWindow addRootViewControllerViewIfPossible] + 58
  7   UIKit                               0x0000000100293c70 -[UIWindow _setHidden:forced:] + 282
  8   UIKit                               0x000000010029cffa -[UIWindow makeKeyAndVisible] + 51
  9   ControlStyleDemo                    0x00000001000045e0 -[AppDelegate application:didFinishLaunchingWithOptions:] + 672
  10  UIKit                               0x00000001002583d9 -[UIApplication _handleDelegateCallbacksWithOptions:isSuspended:restoreState:] + 264
  11  UIKit                               0x0000000100258be1 -[UIApplication _callInitializationDelegatesForURL:payload:suspended:] + 1605
  12  UIKit                               0x000000010025ca0c -[UIApplication _runWithURL:payload:launchOrientation:statusBarStyle:statusBarHidden:] + 660
  13  UIKit                               0x000000010026dd4c -[UIApplication handleEvent:withNewEvent:] + 3189
  14  UIKit                               0x000000010026e216 -[UIApplication sendEvent:] + 79
  15  UIKit                               0x000000010025e086 _UIApplicationHandleEvent + 578
  16  GraphicsServices                    0x0000000103aca71a _PurpleEventCallback + 762
  17  GraphicsServices                    0x0000000103aca1e1 PurpleEventCallback + 35
  18  CoreFoundation                      0x00000001018d3679 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__ + 41
  19  CoreFoundation                      0x00000001018d344e __CFRunLoopDoSource1 + 478
  20  CoreFoundation                      0x00000001018fc903 __CFRunLoopRun + 1939
  21  CoreFoundation                      0x00000001018fbd83 CFRunLoopRunSpecific + 467
  22  UIKit                               0x000000010025c2e1 -[UIApplication _run] + 609
  23  UIKit                               0x000000010025de33 UIApplicationMain + 1010
  24  ControlStyleDemo                    0x0000000100006b73 main + 115
  25  libdyld.dylib                       0x0000000101fe95fd start + 1
  26  ???                                 0x0000000000000001 0x0 + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

现在，我们怀疑出错的地址是0x0000000100004af8（可以根据执行文件名判断，或者最小的栈地址）。为了进一步精确定位，我们可以输入以下的命令：

```
image lookup --address 0x0000000100004af8
```
命令执行后返回：

```
Address: ControlStyleDemo[0x0000000100004af8] (ControlStyleDemo.__TEXT.__text + 13288)
Summary: ControlStyleDemo`-[RootViewController viewDidLoad] + 312 at RootViewController.m:53
```
我们可以看到，出错的位置是RootViewController.m的第53行。



##应用发布后崩溃收集
###Xcode自带线上应用崩溃日志
在Xcode中查看具体崩溃信息
在**Xcode**中菜单的**Window**下选择**Organizer**,在打开的窗口中选择**Crashes**，这样Xcode会开始下载相关的崩溃信息到本地中，可以在左侧的窗口中切换线上版本，点击右侧**Open in Project**能直接定位到代码。

###根据crash崩溃报告的内存地址定位到代码位置
[Understanding and Analyzing iOS Application Crash Reports](https://developer.apple.com/library/ios/technotes/tn2151/_index.html)

1，首先要有崩溃的app上传时候的打包文件，也就是 .xcarchive文件。这个文件可以通过以下方法找到，点击Xcode右上角的Organizer，然后点击Organizer上面的Archives,就可以看到下面有个列表，列出的都是打包的文件，其中一个就是你打包时候留下的，找到它。找到后点击右键显示包内容，看到dSYMs文件和Products文件夹 。先打开dSYMs文件夹，看到yourapp.app.dSYM文件，为了方便，把它复制到桌面。然后回去打开Products文件夹看到Applications文件夹，打开之，然后看到了你的app,同样把他复制到桌面。到这里准备工作完成

2,对着复制到桌面的yourapp.app.dSYM文件右键，显示包内容，然后是Contents文件夹，进入有Resources文件夹，打开后有DWARF，然后打开终端（在实用工具里）用cd命令打开DWARF文件夹 回车

3，然后输入xcrun atos -arch armv7 -o  GoddessPlan0xc3812 0x000a9812  
红色部分根据打包编译情况可以是 armv6,armv7,armv7s
绿色部分是你的app名字
蓝色部分是crash的内存地址，

###Umeng Debug Tool
我们洋葱数学中采用了umeng的错误收集工具来收集错误日志，umeng提供了一个简单的工具来实现代码定位，原理同上一段（**根据 crash 崩溃 报告的内存地址定位到代码位置**）。

[Umeng 错误调试tool 连接](http://www.umeng.com/umeng30_error_type)

[使用简单说明](http://www.umeng.com/umeng30_error_type)


###Apple一些关于Debug的官方文档
<https://developer.apple.com/support/debugging/cn/>

<https://developer.apple.com/support/debugging/>

<https://developer.apple.com/library/ios/technotes/tn2239/_index.html>

[Understanding and Analyzing iOS Application Crash Reports](https://developer.apple.com/library/ios/technotes/tn2151/_index.html)

[How to Match a Crash Report to a Build](https://developer.apple.com/library/ios/qa/qa1765/_index.html)