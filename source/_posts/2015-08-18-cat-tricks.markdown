---
layout: post
title:  "Unix cat tricks"
category: news
author: "diggzhang"
---

最近有一个有意思的需求，合并我们的所有代码，生成一份word文档，保留文档的前30页，后30页，每页45行，而且每页不能有空行。既然说起合并，首先想到了cat。

### OK!先合并所有文件

```
    $ cat ./matrix/* >> matrixMerge.txt 
```

 这样写乍一看没有什么大问题，几分钟中，CPU温度飙到60摄氏度，我知道不大对劲了，仔细一看，磁盘也报满。然后发现已经生成了一个198G的庞然大物。

```
......
drwxr-xr-x   4 zz staff   136B  8 25 08:44 lib
-rw-r--r--   1 zz staff   198G  8 25 09:25 matrixMerge.txt
-rw-r--r--   1 zz staff    37B  8 25 08:44 nodemon.json
.....

```

好吧，没那么简单(你读这句有没有脑补着唱出来)，

### 有选择的合并

妥协一下，有选择的合并：

```
    $ cat ./matrix/*.js >> matrxMerge.txt
```

这下貌似没有问题了，春风得意着，翻开文件看，还有各种空行啊！！！！！！不想要啊！！！！！！尼这是闹肾呢！！！！！！！！！还能不能愉快的玩耍了！！！！！！！

### sed

那好，字符匹配去空行，首先想到的是awk和sed。用sed的正则可以轻松匹配出空行。

```
    $ cat -s ./matrix/*.js | sed '/^[[:space]]*$/d' >> mergeWithoutBlank.txt
```

sed大法好。

### head tail

后面简单了，头尾30页，每页45行。tail和head可以抽取指定行。

```
    $ head -n 1350 merge.txt
    $ tail -n 1350 merge.txt
```

导入到word，大功告成。

### Word 

怎么可能成呢……导入word后生成了40多页文档，而且发现word限定每页42行。还需要调整排版。

```
    1. 行间距设置跨度 15.5 磅
    2. 调整上下页边距1.9cm，纸型A4
    3. 字体设置（完成宋体+五号设置）
    4. 文字对齐字符网格   字符:每行（40）每页（45)
```

