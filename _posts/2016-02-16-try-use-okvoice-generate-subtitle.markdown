---
layout: post  
title:  "使用okvoice制作视频字幕的可行性分析"  
category: news  
author: "yinxin630"
---

[okvoice][1]提供了免费的语音识别接口，以及收费的字幕大师软件。经过尝试使用之后，得出结论：okvoice的语音识别准确度过差，生成的字幕仍需要大量的人力成本来更正，并不比直接由人力制作的效率高。

## okvoice接口测试

这是[语音识别的文档][2]，经过对接口进行测试，发现实际接口跟文档有较大出入，（文档的接口URL都写错了~~）。联系客服（工程师14号放假不在），得知实际的接口URL为`http://api.okvoice.com/asr/api`；文档中描述format为可选字段，实测发现必须加上这个字段；文档中的错误与实际获得的错误码内容不匹配，不过影响不大仍可读；期间一段时间接口调用均返回`asr fail`的结果，通知客服后等待一段时间后恢复正常。

按照文档的说明，编写了下面的程序来生成符合接口规则的请求地址：

```
'use strict'

const Http = require('http');
const Fs = require('fs');
const Md5 = require('md5');
const Co = require('co');
const Promisify = require('es6-promisify');
const Crypto = require('crypto');

Co(function* (){
    let voice = yield Promisify(Fs.readFile)('./voice.wav');
    
    const requestParamStr = `apiKey=a0764790135a124cdd9a91e47a8e22c6&expires=${new Date().getTime() + 1000 * 60 * 60 * 24}&format=wav&lang=zh-CN&md5=${Md5(voice)}`;
    const hash = Crypto.createHmac('sha1', 'c3a9f1a337c7c3e47e8339a6b7fd0dd9').update(requestParamStr).digest('hex');
    
    console.log(`http://api.okvoice.com/asr/api?${requestParamStr}&signature=${hash}`);
});
```

使用音频录制软件，录制一段洋葱视频的语音，使用ffmpeg将语音转换成符合okvoice要求的格式，（16khz、16bit、单声道），运行程序获取请求URL：

```
$ node app.js
http://api.okvoice.com/asr/api?apiKey=a0764790135a124cdd9a91e47a8e22c6&expires=1455674840646&format=wav&lang=zh-CN&md5=fc3a25051e8b82312826f57b6c629108&signature=5f0b6b00ee3497e818d62bef2e0fb239fca21fb2
```

使用postman调用接口，得到结果：

* 文件大小：844kb （接口限制1MB）
* 接口响应时间：17825ms
* 识别结果：

```
所以和远的距离相同的点一定是做一个又一个说一个又一个城卫队出现那么小凡说也就是成堆出现了道道说这么多多上还有一个说被我们漏掉了那就是玲珑作为分界点兵是一个特例因为只有林叶说所以呢他的相反说就是他自己但是表达的时候我们仍然不能说灵石想
```
* 原文为：

```
所以，和原点距离相同的点，一定是左一个右一个左一个右一个，成对出现，那么相反数也就是成对出现了。等等，说了这么多，数轴上还有一个数被我们漏掉了，那就是零。作为分界点，零是一个特例，因为只有零既不是正数也不是负数，所以呢，它的相反数就是它自己，但是表达的时候，我们仍然不能说零是相反数。
```

分析结果得出，识别出的内容没有断句，且误差较大，没有二次加工的价值。

## 字幕大师软件测试

[字幕大师][3]提供了直接将视频提取出字幕的服务，[收费价格][4]为按视频时长每1小时40元。软件为Windows系统应用，在xp虚拟机中安装上应用，注册应用帐号，默认包含20元余额可用于测试。

测试视频为负数那一节，测试结果图：
![](/assets/resource/try-okvoice/okvoice.png)

生成的SRT字幕文件：[字幕文件](/assets/resource/try-okvoice/example.txt)

## 综合结论

okvoice对于加快字幕生成工作效率并无太多价值，不具备可行性。


  [1]: http://www.okvoice.com/
  [2]: http://dev.okvoice.com/asr.php
  [3]: http://oksrt.okvoice.com/
  [4]: http://oksrt.okvoice.com/price_list.html