---
layout: post  
title:  "Appium Test Case-Python篇"  
category: tech
tags: 
    CI,Appium
author: "LiuZhi"
comments: true
toc: true

---

#### Appium的Client/Server结构

appium的核心其实是一个暴露了一系列REST API的server

这个server的功能其实很简单：监听一个端口，然后接收由client发送来的command。翻译这些command，把这些command转成移动设备可以理解的形式发送给移动设备，然后移动设备执行完这些command后把执行结果返回给appium server，appium server再把执行结果返回给client。

在这里client其实就是发起command的设备，一般来说就是我们代码执行的机器，执行appium测试代码的机器。狭义点理解，可以把client理解成是代码，这些代码可以是java/ruby/python/js的，只要它实现了webdriver标准协议就可以。

#### Appium-Python-Client

appium client是对webdriver原生api的一些扩展和封装。它可以帮助我们更容易的写出用例，写出更好懂的用例，并且它是配合原生的webdriver来使用的，因此二者必须配合使用缺一不可。

最推荐的安装方法:

```
pip install Appium-Python-Client
```

#### HTMLTestRunner

[下载传送门](http://tungwaiyip.info/software/HTMLTestRunner.html)

HTMLTestRunner是Python标准库中单元测试模块的扩展，它生成易于使用的html测试报告

下载HTMLTestRunner.py文件后，把HTMLTestRunner文件放到Python27\Lib的目录下即可

具体使用方法见文章末尾处的完整用例代码

#### Appium的Session

session就是一个会话，在webdriver/appium，你的所有工作永远都是在session start后才可以进行的。一般来说，通过POST /session这个URL，然后传入Desired Capabilities就可以开启session了。

开启session后，会返回一个全局唯一的session id，以后几乎所有的请求都必须带上这个session id，因为这个seesion id代表了你所打开的浏览器或者是移动设备的模拟器。

#### Appium的Desired Capabilities

Desired Capabilities在启动session的时候是必须提供的。

Desired Capabilities本质上是key value的对象，它告诉appium server这样一些事情：

* 本次测试是启动浏览器还是启动移动设备？
* 是启动andorid还是启动ios？
* 启动android时，app的package是什么？
* 启动android时，app的activity是什么？

Appium的Desired Capabilities是扩展了webdriver的Desired Capabilities的，下面是一些通用配置：

* automationName：使用哪种自动化引擎。appium（默认）还是Selendroid？
* platformName：使用哪种移动平台。iOS, Android, orFirefoxOS？
* deviceName：启动哪种设备，是真机还是模拟器？iPhone Simulator, iPad Simulator, iPhone Retina 4-inch, Android Emulator, Galaxy S4, etc...
* app：应用的绝对路径，注意一定是绝对路径。如果指定了appPackage和appActivity的话，这个属性是可以不设置的。另外这个属性和browserName属性是冲突的。
* browserName：移动浏览器的名称。比如Safari' for iOS and 'Chrome', 'Chromium', or 'Browser' for Android；与app属性互斥。

下面的属性是android平台特定的：

* appActivity：待测试的app的Activity名字。比如com.yangcong345.android.phone.presentation.activity.SplashActivity
* appPackage：待测试的app的java package。比如com.yangcong345.android.phone
	
#### 控件定位

在appium的定位方法中，下面这些方法是可以为我们使用的。也就是说，我们通过下面几个约定好的方式，按照webdriver和appium的DSL（自行搜索并理解）进行控件特征的描述和定位。

* find by "class" (ui component type，andorid上可以是android.widget.TextView)
* find by "xpath" (an abstract representation of a path to an element, with certain constraints，由于appium的xpath库不完备的原因，这个不太推荐)
* find by "id"(android上是控件的resource id)
* find by name(如android的Button的名字)

定位方法代码展示:

```
self.driver.find_elements_by_class_name('android.widget.TextView')

self.driver.find_element_by_xpath("//android.widget.LinearLayout[1]/android.widget.FrameLayout[1]/android.view.ViewGroup[1]/android.widget.FrameLayout[1]/android.widget.FrameLayout[1]/android.widget.LinearLayout[1]/android.widget.LinearLayout[1]/android.widget.LinearLayout[1]/android.widget.Button[1]")

self.driver.find_element_by_id('com.yangcong345.android.phone:id/rbStudent')

self.driver.find_element_by_name(u'人教版')
```

#### 常用方法

[官方Demo传送门](https://github.com/appium/sample-code/tree/master/sample-code/examples/python)

当定位到一个控件或是一个控件组的时候（find_element跟find_elements的区别），就可以对控件执行一些事件动作，比如点击，滑动等

代码展示:

```
self.driver.find_element_by_id('com.yangcong345.android.phone:id/rbStudent').click()

els = el.find_elements_by_class_name('android.widget.TextView')
self.driver.scroll(els[len(els) - 1], els[0])
```

**重点:进度条如何定位到末尾**

```
el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/mediacontroller_progress')
self.assertIsNotNone(el)

end = el.size.get('width')
y = el.location.get('y')
action = TouchAction(self.driver)
action.tap(None, end + 95, y).perform()//此处tap方法有个bug，就是首个参数需要传None才能使action的绝对定位生效
```

还有常用的方法有：

* 输入框输入字符
* 获取控件的属性
* 一些断言方法 

代码展示:

```
self.driver.find_element_by_id('com.yangcong345.android.phone:id/etUserName').send_keys('qq3@qqq.com')

el.get_attribute('checked')//返回的是字符串

self.assertEqual(el.get_attribute('checked'), 'true')
self.assertIsNotNone(el)
//还有很多断言方法在此不列举了
```

#### 完整用例

```
# coding=utf-8
import os
import unittest, sys, time, re, datetime
import HTMLTestRunner
from appium import webdriver
from time import sleep
import sys
from appium.webdriver.common.touch_action import TouchAction
from appium.webdriver.common.multi_action import MultiAction

reload(sys)
sys.setdefaultencoding('utf-8')

cwd = os.getcwd()
phone_student = 'erwa@qq.com'


class SimpleAndroidTests(unittest.TestCase):
    def setUp(self):
        desired_caps = {}
        desired_caps['appium-version'] = '1.0'
        desired_caps['platformName'] = 'Android'
        desired_caps['platformVersion'] = '5.0.1'
        desired_caps['deviceName'] = '192.168.56.101'
        desired_caps['app'] = os.path.abspath(cwd + '/YCMath_Android_V2.7.0_guanghetv.apk')
        desired_caps['appPackage'] = 'com.yangcong345.android.phone'
        desired_caps['appActivity'] = 'com.yangcong345.android.phone.presentation.activity.SplashActivity'

        self.driver = webdriver.Remote('http://127.0.0.1:4723/wd/hub', desired_caps)
        self.driver.implicitly_wait(60)

    def tearDown(self):
        # end the session
        self.driver.quit()

    def input_name_pwd(self, name, pwd):
        self.driver.find_element_by_id('com.yangcong345.android.phone:id/etUserName').send_keys(name)
        self.driver.find_element_by_id('com.yangcong345.android.phone:id/etPwd').send_keys(pwd)

    def login(self):
        self.driver.find_element_by_id('com.yangcong345.android.phone:id/btn_login').click()
        self.input_name_pwd(phone_student, '123456')
        self.driver.find_element_by_id('com.yangcong345.android.phone:id/btnLogin').click()
        sleep(3)

    def open_chapter(self):
        self.driver.find_element_by_id('com.yangcong345.android.phone:id/mask_bottom').click()
        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/tabs')
        self.assertIsNotNone(el)
        els = el.find_elements_by_class_name('android.widget.TextView')
        self.driver.scroll(els[len(els) - 1], els[0])
        sleep(1)
        el = self.driver.find_element_by_name(u'不等式与不等式组')
        self.assertIsNotNone(el)
        el.click()
        sleep(3)

    def open_theme(self):
        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/exp_lv')
        self.assertIsNotNone(el)
        el = self.driver.find_element_by_name(u'不等式的基本概念')
        self.assertIsNotNone(el)
        el.click()
        sleep(3)

    def open_topic(self, name):
        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/recycler_view')
        self.assertIsNotNone(el)
        el = self.driver.find_element_by_name(name)
        self.assertIsNotNone(el)
        el.click()
        sleep(3)

    def close_small_video(self):
        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/btn_close')
        self.assertIsNotNone(el)
        el.click()

    def seek_to_end(self):
        self.close_small_video()

        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/mediacontroller_progress')
        self.assertIsNotNone(el)

        end = el.size.get('width')
        y = el.location.get('y')
        action = TouchAction(self.driver)
        action.tap(None, end + 95, y).perform()
        sleep(3)

        self.close_small_video()

    def click_rest(self):
        el = self.driver.find_element_by_name(u'先休息一下')
        self.assertIsNotNone(el)
        el.click()

    def module_has_video(self):
        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/recycler_view')
        self.assertIsNotNone(el)
        el = el.find_element_by_name(u'视频讲解')
        self.assertIsNotNone(el)

    def open_module_video(self):
        el = self.driver.find_element_by_id('com.yangcong345.android.phone:id/recycler_view')
        self.assertIsNotNone(el)
        el = el.find_element_by_name(u'视频讲解')
        self.assertIsNotNone(el)
        el.click()

    def is_at_theme(self):
        el = self.driver.find_element_by_name(u'完成度')
        self.assertIsNotNone(el)

    def test_3_1_3(self):
        self.login()
        self.open_chapter()
        self.open_theme()
        self.open_topic(u'不等式引入')
        self.module_has_video()
        self.open_module_video()
        self.seek_to_end()
        self.click_rest()
        self.is_at_theme()


if __name__ == '__main__':
    suite = unittest.TestSuite()
    suite.addTest(
        unittest.defaultTestLoader.loadTestsFromTestCase(SimpleAndroidTests)
    )
    timestr = time.strftime('%Y%m%d%H%M%S', time.localtime(time.time()))
    filename = os.path.abspath(cwd + '/result') + "/result_" + timestr + ".html"
    print (filename)
    fp = open(filename, 'wb')
    runner = HTMLTestRunner.HTMLTestRunner(
        stream=fp,
        title='测试结果',
        description='测试报告'
    )
    runner.run(suite)
    fp.close()  # 测试报告关闭

```
