---
layout: post
title:  "Mac环境下如何配置Appium"
category: tech
author: "lucien"
---


#一：下载（appium-1.4.13.dmg）
地址（1）<https://github.com/appium/appium/releases>

地址（2）<https://bitbucket.org/appium/appium.app/downloads/> 
#二：配置环境
###1、安装Homebrew(终端下运行下面代码)

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
查看版本（检测是否成功安装)

```
brew -v
```
###2、安装Node

```
brew install node
```
查看版本（检测是否成功安装)

```
node -v
```
#三:检测appium是否成功配置好
###1.点击红圈内按钮 

![](/assets/images/appium-config/appium-config-3-1.png)

运行后

```
Running iOS Checks
✔ Xcode is installed at /Applications/Xcode.app/Contents/Developer
✔ Xcode Command Line Tools are installed.
✔ DevToolsSecurity is enabled.
✔ The Authorization DB is set up properly.
✔ Node binary found at /usr/local/bin/node
✔ iOS Checks were successful.
Running Android Checks
✔ ANDROID_HOME is set to "/Users/guokr/Downloads/adt/sdk"
✖ JAVA_HOME is not set
```
###2.可能出现的问题

iOS环境:

`Xcode Command Line Tools未安装.`

解决办法:

```
xcode-select --install
```
Android环境:(配置好JAVA环境,下载好SDK)

`android or java 的HOME is not set.`

解决办法:
自己建立一个文件,名字为: `.bash_profile`

vim编辑

```
export ANDROID_HOME=/Users/yulu/Library/Android/sdk

export PATH=$PATH:$ANDROID_HOME/platform-tools

export PATH=$PATH:$ANDROID_HOME/build-tools

export PATH=$PATH:$ANDROID_HOME/tools

export JAVA_HOME=$(/usr/libexec/java_home)
```
然后保存
###3.设置好后,再按照(1)的操作进行.出现下图证明成功安装
![](/assets/images/appium-config/appium-config-3-3.png)