---
layout: post
title:  "onionBeep洋葱应急工作台"
category: news 
author: "diggzhang"
---

服务器挂掉后抢修是个充满乐趣但紧张繁琐的过程，期间需要随时监控各项参数协助排错，不得不在不同的控制台之间频繁切换，**onionBeep**为简化这一过程而生。名字起得太屌了，说白就是tmux加个plugin，然后自己写了个[tmuxinator](https://github.com/diggzhang/tmuxinator.git)的配置脚本。

###Package

- ruby
- gem
- tmux
- [tmuxinator](https://github.com/diggzhang/tmuxinator.git) (`sudo gem install tmuxinator
`)

###Setup

保证系统环境里有上述的工具后，首先与服务端交换`ssh-key`,保证ssh不询问登录密码。

```
  生成ssh pub key
  $ ssh-keygen
  拷贝到目标服务器
  $ ssh-copy-id -i ~/.ssh/id_rsa.pub 10.9.9.9
```
交换ssh-key成功后，ssh时无需再输入登录密码。拷贝[tmuxinator](https://github.com/diggzhang/tmuxinator.git)项目里yamlfile中的文件到目标服务器mux配置目录`$HOME/.tmuxinator/`。初次部署要记得单独配置一下yamlfile。
 

###Basic useage

启动beep：

```
tmuxinator start <ymalfilename>
tmuxinator start node1
tmuxinator start node2
```

tmux切换屏幕 `Control + b 后按 n`
tmux切换pane `Control + b 后按 o`


