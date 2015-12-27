---
layout: post
title:  "iOS开发常见问题"
category: tech
author: "jichengsun"
---

#线程异步问题
###tableview reloadData 
  
  tableView reloadData 属于主线程异步执行方法，当有些代码需要确保在tableView reloadData完成后再执行时，可以使用GCD方式将代码同样放置到主线程异步执行
   
```objective-c
   dispatch_async(dispatch_get_main_queue(), ^{
   		//your code
   })
```
   
###UITableView Always change the dataSource and reloadData in the mainThread.
   
   
###uiview layoutSubviews
   在uiview初始化后 layoutSubviews不会立即执行，中间可能存在时间差，因此若有需要layoutSubviews执行后，再执行的代码，也可同样用上面的方式
   
```objective-c
   dispatch_async(dispatch_get_main_queue(), ^{
   		//your code
   })
```
   e.p:洋葱数学中，交互视频出选项，当HMSideMenu的layoutSubviews未执行，就显示选项时，会造成第一个选项无法出现的Bug
   
###extern const static

```objective-c
const NSString *HSCoder1 = @"111";//*HSCoder"不能被修改， "HSCoder"能被修改
NSString const *HSCoder2 = @"222";//"*HSCoder"不能被修改， "HSCoder"能被修改
NSString * const HSCoder3 = @"333";//"HSCoder"不能被修改，"*HSCoder"能被修g改
```

**extern**如果想要一个全局常量，就需要使用extern,例如在.h中 

```objective-c
extern NSString * const str;
```

在.m中:

```objective-c
NSString * const str = @"aaa";
```

