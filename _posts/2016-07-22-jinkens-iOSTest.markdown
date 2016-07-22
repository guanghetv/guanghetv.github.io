---
layout: post
title:  "Jinkens+iOS测试"
category: tech
author: "lucien"
---


#一：安装Jinkens
<http://linjunpop.logdown.com/posts/162202-set-up-jenkins-server-on-the-mac-mini-to-run-ios-tests>


#二:构建iOS工程
```
项目名称:给项目工程起名字.
描述:写一写关于项目的介绍.
GitHub project
        Project url : 项目的github地址
丢弃的构建
    strategy    Log Roraton             
    保持构建天数  5     保持最大构建天数  10
    
源码管理
     Git
  Repositories      Repository URL  https://github.com/guanghetv/YCMath345-iOS.git   (仓库地址)
  Credentials  XXXXXXXXXXX@qq.com    XXXXXXXXX     (github账号密码)
  Branches to build  branch Specifier    */develop-3.5.0   (分支)

构建环境
Provide Node & npm bin/ folder to PATH    node.js     (配置node.js,供测试用例)

构建
    Xcode 
    
    General build setting 
    Target  :   YCMath345-iOS     
    Clean before build ?    Yes  
    Configuration   Debug 
    
Pack application and build .ipa ?

.ipa filename pattern YCMath345-iOS 
Output directory $(WORKSPACE)/Release

Advanced Xcode build options
Clean test reports?    Yes
Custom xcodebuild arguments :  -sdk iphonesimulator (这个参数是个坑,配了好长时间,因为appium只能运行iphonesimulator版,不能运行iphoneos版)
Xcode Workspace File    /Users/yulu/.jenkins/workspace/YCMathiOSCI/YCMath345-iOS
Xcode Project File          /Users/yulu/.jenkins/workspace/YCMathiOSCI/YCMath345-iOS
Build output directory     ${WORKSPACE}/build


Execute Shell
Command  mocha ${WORKSPACE}/script/AppiumTest-iOS.js


构建后步骤
Upload  to fir.im 
fir.im token   (输入在fir.im的token)
IPA/APK Files (optional)  /Users/yulu/.jenkins/workspace/YCMathiOSCI/Release
ExInclude IPA/APK File Name    **/*unaligned*.apk,**/*unsigned*.apk
Build Notes   Updated by yulu with fir-jenkins


Email Notification
Recipients  yulu@gaunghe.tv    (测试成功或失败都会发邮件通知)
``` 
#3.总结
根据以上操作,就可以实现Github+Jinkens+Appium+Fir.im+Email.
即每向Github提交一次代码,Jenkins都会拉取代码到本地然后编译,打包,再交由Appium跑测试用例,测试成功后,Jenkins将打包的文件上传到Fir.im上,并发邮件通知开发者.