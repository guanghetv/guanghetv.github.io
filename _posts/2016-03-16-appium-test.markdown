---layout: post
title: "Appium+Eclipse编写测试用例"
category: tech
autor: "lucien"
---

#一:环境准备好.
*详情见第一篇文章(Mac环境下如何配置Appium环境.*
#二:使用eclipse直接创建案例工程

1、打开Eclipse，【file】-->【New】--> 【Project】

2、选择【Java Project】--> 【Next】

3、输入工程名 AppiumDemo，点击【finish】

4、右键点击工程 New-Floder,新建两个文件夹：apps和libs.目录结构如下：

![](/assets/images/appium-test/appium-test-2-4.png)
#三：导入测试的类库
1、导入selenum类库：
下载地址：<http://docs.seleniumhq.org/download/>

 `(1) <sselenium-server-standalone-2.44.0.jar>`
 
  `(2) <selenium-java-2.44.0.zip>`
  
2、导入appium类库：
下载地址：<http://appium.io/downloads.html>

 `1)java-client-1.2.1.jar`  (最好是1.3版本)

3、右键点击工程空白处，选择【Build Path】-->【Config Build Path】  把类库导入到工程
#四：配置apk或app

Android：把你的apk放到apps目录下。

iOS：把你的app放到apps目录下。

#五：建立package包和案例文件

1、在src文件夹右键单击，【New】-->【package】，输入包名：com.yulu.demo,点击【Finish】.

2.在package下新建类：【New】-->【class】,输入名字：AppiumTest，点击【finish】

```
package com.yulu.demo;
//导入包类
import io.appium.java_client.AppiumDriver;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities; 
 
import java.io.File;  
import java.net.URL;
import java.sql.Time;
import java.util.List;
import java.util.concurrent.TimeUnit;
import java.sql.Time;
//准备测试
public class YCMathTest {
	//初始化AppiumDriver
          private AppiumDriver driver;
    @Before
    public void setUp() throws Exception {
    	
    //设置APP路径
    File classpathRoot = new File(System.getProperty("user.dir"));
    File appDir = new File(classpathRoot,"apps");
    File app = new File(appDir, "YCMath345-iOS.app");
    ///Users/sks/Library/Developer/Xcode/DerivedData/YCMath345-iOS-gswmmorclgkffeevdytydhjkdhjk/Build/Products/Debug-iphonesimulator/YCMath345-iOS.app
    System.out.println("设置路径完毕");
    //设置自动化相关参数
    DesiredCapabilities capabilities = new DesiredCapabilities();
    capabilities.setCapability("appium-version", "1.1.0");
    capabilities.setCapability("platformVersion", "9.2");
    capabilities.setCapability("platformName", "ios");
    capabilities.setCapability("deviceName", "iPhone 6");
    System.out.println("设置自动化相关参数");
    //设置apk路径
    capabilities.setCapability("app", app.getAbsolutePath());
    //如果测试的是AndroidApp的话,需要设置app的主包名和主类名-------iOS可以省略
//    capabilities.setCapability("appPackage", "com.example.android.contactmanager");
//    capabilities.setCapability("appActivity", ".ContactManager");
          
    //初始化 AppiumDriver
    driver = new AppiumDriver(new URL("http://127.0.0.1:4723/wd/hub"),capabilities );
    //设置等待秒数
    driver.manage().timeouts().implicitlyWait(60, TimeUnit.SECONDS);
    System.out.println("初始化 AppiumDriver");
    }
  //运行完成后
    @After
      public void tearDown() throws Exception{
    	
//    	driver.wait(12000);
    	driver.quit();
    	System.out.println("运行结束!!!!!!!");
    	System.out.println("即将开启下一个Session");
       }
    
    
//------测试用例编写--------//
    //Session One
    @Test
    public void AppiumTestOne() throws InterruptedException{
    	System.out.println("Session One-First case-Start");
	//driver.findElement(By.name("马上开始")).click();
	//driver.findElement(By.xpath("//UIAApplication[1]/UIAWindow[1]/UIAButton[1]")).click();
    //  WebElement el = ((WebElement) driver.findElementsByIosUIAutomation("马上开始"));
	//  el.click();
    	
      driver.findElement(By.name("马上开始")).click();
      System.out.println("点击马上开始!");
	
	  driver.findElement(By.name("人教版")).click();
	  System.out.println("点击人教版,选择教材!");
	  
	  driver.findElement(By.name("七年级上")).click();
	  System.out.println("点击七年级上,选择年纪!");
	  
	  
	  driver.findElement(By.name("引入")).click();
	  System.out.println("点击引入,跳转到视频界面!");
	  
	  driver.findElement(By.name("yc coachmark")).click();
	  System.out.println("点击蒙版,接下来播放视频");
	  
	  System.out.println("设置了线程休眠20秒....");
	  //线程睡眠,ms
	  Thread.sleep(20000);
	  //driver.manage().timeouts().implicitlyWait(20, TimeUnit.SECONDS);
      System.out.println("视频播放20秒后");
	 // driver.wait(10);
	  System.out.println("Session One-First case-End!");
}
    
  //Session Two
    @Test
    public void AppiumTestTwo() throws InterruptedException{
    	System.out.println("Session Two-first case-start");
    	//返回按钮
    	driver.findElement(By.name("注册或登录")).click();
    	System.out.println("点击注册或登录!");
    	
    	driver.findElement(By.name("学生")).click();
    	System.out.println("点击button,身份为学生!");
    	 
      //  driver.findElement(By.name("手机号//邮箱")).sendKeys("15725040279");;
      //  System.out.println("输入手机号或邮箱--by.name");
        
        driver.findElement(By.xpath("//UIAApplication[1]/UIAWindow[1]/UIATextField[1]")).sendKeys("15725040279");;
        System.out.println("输入手机号或邮箱!");
        
        driver.findElement(By.xpath("//UIAApplication[1]/UIAWindow[1]/UIASecureTextField[1]")).sendKeys("yulu83741319");
        System.out.println("输入密码!");
        
		driver.findElement(By.xpath("//UIAApplication[1]/UIAWindow[5]/UIAKeyboard[1]/UIAButton[4]")).click();
        System.out.println("点击键盘完成输入!");
		
        driver.findElement(By.name("立即登录")).click();
        System.out.println("点击了,立即登录按钮.");
        
        Thread.sleep(10000);
        System.out.println("睡眠10秒");
        
        System.out.println("Session Two-first case-End!");
    }
}
```
Android的测试用例与iOS的几乎一样,只需要改一下:

名称,路径,自动化相关参数

添加:

主包名,主类名.(iOS测试用例中已注释)

#五：启动Appium GUI

①iOS版

1.配置app的路径，模拟器的机型版本等等。

![](/assets/images/appium-test/appium-test-5-1.png)

2.设置appium的地址，端口等基本信息。

![](/assets/images/appium-test/appium-test-5-2.png)

3.点击Launch

**`必须先运行appium才可以进行测试。`**

六：测试开始

1.回到eclipse，右键单击AppiumDemo工程 【Run as】-->【JUnit Test】

测试开始。

顺利完成的话会提示successful.