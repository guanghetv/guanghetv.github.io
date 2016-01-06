---
layout: post  
title:  "在2015年，WEB页面平均体积增长了16%"  
category: news  
author: "yinxin630"
---

原文：[点击打开][1]
作者：[碎碎酱][2]

这种情况又发生了，[HTTP Archive Report][3]统计了互联网上50万个受人欢迎的网站，最终得出结论：2015年平均页面体积增长了16%，达到了2262KB！在[2014年也有着类似的情况发生][4]。

下面的数据分析来自一些公开的网站和购物网站，它不能分析类似FaceBook这种需要登录的网站。

<table summary="average page weight, December 2015" style="text-align:right !important;margin:20px auto">
<tbody><tr>
<th>technology</th>
<th>end 2014 (KB)</th>
<th>end 2015 (KB)</th>
<th>increase (%)</th>
</tr>
<tr>
<th>HTML</th>
<td>59</td>
<td>66</td>
<td>12%</td>
</tr>
<tr>
<th>CSS</th>
<td>57</td>
<td>76</td>
<td>33%</td>
</tr>
<tr>
<th>JavaScript</th>
<td>295</td>
<td>363</td>
<td>23%</td>
</tr>
<tr>
<th>Images</th>
<td>1,243</td>
<td>1,443</td>
<td>16%</td>
</tr>
<tr>
<th>Flash</th>
<td>76</td>
<td>53</td>
<td>-30%</td>
</tr>
<tr>
<th>Other</th>
<td>223</td>
<td>261</td>
<td>17%</td>
</tr>
<tr>
<th>Total</th>
<td>1,953</td>
<td>2,262</td>
<td>16%</td>
</tr>
</tbody></table>

HTML的内容增长了7KB，是因为每篇文章都长了12%吗？我想肯定不是。

不出意料的，Flash减少了30%, 23KB，仍有五分之一的网站在使用Flash，并以每年26%的速度在衰减。

CSS增长了19KB，有一些小理由来解释这种情况：

1. CSS的能力一直在提升，我们使用它实现页面效果、动画和响应式设计，这在几年前是无法实现的。
2. 像Sass、Less和Stylus这样的预加载器，它们捆绑css代码使其支持嵌套和属性继承。
3. 构建工具使得使用内置图形资源更容易。

CSS的盒模型特性可以帮助我们减少使用复杂的基于float的布局，这会帮助我们缩小体积。

你可能预料JavaScript代码体积会下降，但是它却增长了68KB达到了363KB，这些大量的代码一部分是框架和库，但我估计更多的应该是社交媒体组件或者广告内容。

其它文件，比如字体和视频，增长了17%, 38KB。

通常情况下，涨幅最大的内容是图片，增长了200KB，占据了总体增长体积的65%。这些图片通常只有一半的内容是被所有页面所需要的，这样全部加载的做法是浪费的。

其它的不足：

* 25%的网站没有使用GZIP压缩
* 101 HTTP file requests are made — up from 95 a year ago
* 页面包含896个DOM元素
* 资源从18个域加载
* 49%的资源是可缓冲的
* 52%的页面使用Google的库，比如Analytics
* 24%的页面没有使用HTTPS
* 36%的页面的资源有4xx和5xx的HTTP错误
* 79%的页面使用了重定向

## 为什么页面体积开始膨胀

有一个简单的而解释：我们正在做一件可怕的工作。

作为一个开发者，我喜爱着WEB，但是作为一个用户，情况通常是相反的，网站通常包含着无孔不入的广告、烦人的弹出窗口、没用的媒体资源和恶意的用户行为追踪，这些内容使人反感。或许这些可以增加暂时的收入，但实际上适得其反。

* 重量级的网站会影响Google搜索引擎的内容优化
* 广告声称是为了内容免费，当用户花费$0.10去浏览一个典型的手机应用站的时候，他们还会在乎它是否免费吗？
* The elevation of ad-blockers to mainstream consciousness during the year highlights user frustrations and the ease at which anyone can abolish irritating content.
* 用户不会等待，[Etsy.com][5]发现页面增加了160KB的图片后，反而导致减少了12%的手机用户量。
* WEB已经开始引起政府机构的注意，例如，UK手机运营商发现使用网络提供服务可以避免用户在各种各样的活动中迷乱。当然，网站的功能必然会变得更为复杂。

内容体积膨胀是令人反感的，没压缩的Shakespeare’s 37 plays有超过80w个单词，需要5MB的流量来下载。考虑下[FaceBook的瞬间文章访问量][6]，提供了另一种渲染页面的方式，页面包含了5个段落，但是你需要耗费额外的3.5M带宽和50MB流量来观看一个3分钟的视频，这是否比莎士比亚著作提供了更多的内容？或许它更漂亮，但是有些时候真的有必要吗？


  [1]: http://www.sitepoint.com/average-page-weight-increased-another-16-2015/
  [2]: http://www.suisuijiang.com
  [3]: http://httparchive.org/trends.phpuisuijiang.com
  [4]: http://www.sitepoint.com/average-page-weight-increases-15-2014/
  [5]: https://www.etsy.com/
  [6]: https://instantarticles.fb.com/