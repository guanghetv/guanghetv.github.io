---
layout: post
title:  "前方高能 how-old.net"
category: news
author: "diggzhang"
---

[How-old.net](http://how-old.net/), 上传你的照片,识别脸部判断你性别和多少岁.

可能你会觉得这功能太low了,老梗,稍等,听我讲完,how-lod.net是一个来自**Microsoft**有意思的故事,最初网站开始内测时候,仅仅期待50个用户尝试一下,可是万万没想到邀请发出后最终全球约有35,000人参与了测试!这玩意儿可是两个人一天搞出来的.
过去的面部识别是用来伤心的,how-old更准,简直堪称ML的黑魔法!

技术点:
***
* 面部识别API [Face Demo](http://www.projectoxford.ai/demo/face),这帮天才用JSON记录了一张脸的数据.
```
"eyeRightBottom": {
    "x": "202.9",
        "y": "109.5"
},
    "eyeRightOuter": {
        "x": "211.5",
        "y": "106.3"
    },
    "noseRootLeft": {
        "x": "161.0",
        "y": "109.4"
    },
```
* 识别面部外,还通过用户访问网站的信息(IP,UA等)以及经纬度,加入算法匹配
```
[ {     "event_datetime": "2015-04-27T01:48:41.5852923Z", 
    "user_id": "91539922310b4f468c3f76de08b15416", "session_id": "fbb8b522-6a2b-457b-bc86-62e286045452", 
    "submission_method": "Search", 
    "face": { "age": 23.0, "gender": "Female" }, 
    "location_city": { "latitude": 47.6, "longitude": -122.3 }, 
    "is_mobile_device": true, "browser_type": "Safari", "platform": "iOS", "mobile_device_model": "IPhone" 
} ] 
```
* ASA
* PowerBI

[Fun with ML, Stream Analytics and PowerBI – Observing Virality in Real Time](http://blogs.technet.com/b/machinelearning/archive/2015/04/29/fun-with-ml-stream-analytics-and-powerbi-observing-virality-in-real-time.aspx)
