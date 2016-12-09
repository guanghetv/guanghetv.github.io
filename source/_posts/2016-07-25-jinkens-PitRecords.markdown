---
layout: post
title:  "Jinkens+踩坑纪录"
category: tech
author: "LiuZhi"
---

# Jenkins

#### Jenkins的通俗概念

jenkins是一个广泛用于持续构建的可视化web工具，说白了就是各种项目的"自动化"编译、打包、分发部署。jenkins可以很好的支持各种语言（比如：java, c#, php等）的项目构建，也完全兼容ant、maven、gradle等多种第三方构建工具，同时跟svn、git能无缝集成，也支持直接与知名源代码托管网站，比如github、bitbucket。
	
#### 安装教程

[传送门](http://linjunpop.logdown.com/posts/162202-set-up-jenkins-server-on-the-mac-mini-to-run-ios-tests)
	
这里需要注意的是：

- 教程里的如下这句不能删除，而是把ip地址改为广播地址，即\<string>--httpListenAddress=0.0.0.0\</string>

> 如果要其他机器也可以访问，把plist里的\<string>--httpListenAddress=127.0.0.1\</string>删掉即可

- 要用命令行来安装，安装包类的安装方法会有意想不到的坑，故不推荐

##### 主要坑介绍

1. 系统管理>系统设置>全局属性

	这里要配置sdk，键：ANDROID_HOME，值：sdk路径
	
	未配置该项时，集成编译过程中会报出未配置ANDROID_HOME环境变量的错，容易误导去配置电脑的系统环境变量

2. 系统管理>系统设置>Jenkins Location

	Jenkins URL：此项是可选的,指定安装Jenkins的HTTP地址,例如http://yourhost.yourdomain/jenkins/. 这个值用来在邮件中生产Jenkins链接.此项是有必要的,因为Jenkins无法探测到自己的URL地址.

3. 系统管理>系统设置>邮件通知

	这里时配置邮件发送服务器，经百般尝试，公司的企业邮箱，并不能顺利生效，推测原因在于outlook邮箱只能用POP和IMAP，而不支持SMTP。若使用163邮箱，可以顺利生效，但经常发送jenkins的邮件，过一大段时间以后，163会以为你老发垃圾邮件，就不让你发了

4. 系统管理>插件管理

	这里就说几个用到的插件
	
	- Git plugin:必装，拉代码用
	- Gradle plugin:必装，编译android项目
	- HTML Publisher plugin:该插件是为了在jenkins web页面上方便的查看appium产生测试用例报告用
	- fir-plugin:[该插件需要手动安装](http://blog.fir.im/jenkins/)，自动发布到fir

5. 系统管理>Global Tool Configuration

	Android持续集成必备环境配置
	
	- JDK
	- Git
	- Gradle

6. 系统管理>脚本命令行
 
		System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts; style-src 'unsafe-inline' *;script-src 'unsafe-inline' *;")
		
7. 项目>配置
	
	- General:这里是基本的无关痛痒的配置
	- 源码管理:这里主要配置下git地址，账户密码，以及要构建的分支、或Tag
	- 构建触发器:这里重点说两个配置
	
		Build periodically:定时触发构建
		
		Poll SCM:定时检查是否有新代码，有则构建
		
		参数：
		
			根据开发需要，假设每一个小时我们需要重新构建一次。选择 Build periodically，在 Schedule 中填写 0 * * * *。第一个参数代表的是分钟 minute，取值 0~59；第二个参数代表的是小时 hour，取值 0~23；第三个参数代表的是天 day，取值 1~31；第四个参数代表的是月 month，取值 1~12；最后一个参数代表的是星期 week，取值 0~7，0 和 7 都是表示星期天。所以 0 * * * * 表示的就是每个小时的第 0 分钟执行一次构建。
			
	- 构建>Invoke Gradle script:
	
		Gradle Version:这里选第五点配置的Gradle版本
		
		Tasks:这里填写常见的gradle 命令，如:clean assemble_guanghetvDebug --stacktrace --debug
		
	- 构建>Execute shell:这里配置appium的测试脚本，如：python script/py261.py，脚本放在了工程目录下的script文件夹下
				
	- 构建后操作>Archive the artifacts:这个配置YCMath/build/outputs/*\*/\*.apk，用于保存生成的apk
	
	- 构建后操作>Publish HTML reports:
		
		HTML directory to archive:script/result,测试报告输出的位置
		
		Index page[s]:index.html,这里要在script/result文件夹中手动添加一个index.html文件，内容如下，这样在jenkins web中可以快捷链接到测试报告列表

			<html>
			<body>
			<script>document.location = '*'</script>
			</body>
			</html> 
	
	- 构建后操作>E-mail Notifation，发送构建邮件
	
	- 构建后操作>Upload to fir.im，自动发布到fir
	
		ExInclude IPA/APK File Name：这里过滤的是**不要上传**的文件
		
			**/*unaligned*.apk,**/*unsigned*.apk
			
	
	







	