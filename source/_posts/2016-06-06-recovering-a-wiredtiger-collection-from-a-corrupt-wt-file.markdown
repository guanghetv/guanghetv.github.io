---
layout: post
title:  "从损坏的wt文件中恢复出WiredTiger集合"
category: tech
author: "cnwelee"
---

常在河边走，哪能不湿鞋。虽然说只要不使用`kill -9`杀进程，一般不会导致MongoDB出问题（Mongo本身有对kill做处理），但是程序总有跑偏的时候，也许哪次服务器重启或者遇到断电之类的，没准就会导致数据库文件损坏。

当然一般的异常关闭后启动不了时可能也就是删除一下lock、pid文件或者tmp下的sock文件即可搞定，根本不是什么问题，偶尔的数据异常--repair也就可以了（数据量大要建一堆索引的时候慎用，等很长时间给你抛出一个修复失败是最容易让人崩溃的……），而且其实开启了journal的情况下非正常关闭mongo时还有比较好用的数据文件自动修复功能，MongoDB的可靠性其实还不错。

### 我们这次要处理的就是一个没有开journal而且还遇到wt数据文件出现异常的数据库。

根据MongoDB的启动日志，`WT_SESSION.open_cursor: unable to read root page from file:collection-3659--4168324323017494102.wt: WT_ERROR: non-specific WiredTiger error`，是3659这个wt文件头出错了，这个要怎么办呢？如果按照某些「专业数据库修复」专家的建议，我们可以把这个损坏的wt文件替换掉。然后你猜会发生什么？`WT_SESSION.open_cursor: collection-3659--4168324323017494102.wt read error: failed to read 4096 bytes at offset 42864640: WT_ERROR: non-specific WiredTiger error`，肯定还会报错嘛，Mongo怎么可能不对文件做校验，暴力地替换肯定行不通啊……再往后怎么发展呢？这就到了专家们的营收环节了，只有到这一步才能体现出他们的『文件修复技术』是有价值的嘛……好了，话不多说，修复个小文件开口就四位数起步，说什么都不能忍……接下来我们自己动手，一步步开启MongoDB数据库修复从入门到放弃系列教程。

以下步骤可以通过直接读取wt文件恢复出对应的集合。关于MongoDB的WT引擎数据的目录布局可以到[这里](http://www.mongoing.com/archives/2214)补习。

# 1.准备工作

## 1.1 wt实用工具包

wt实用工具包是本文用到的核心工具。下面是ubuntu下的操作示例，编译过程嘛，无非就是根据报错信息安装一些必要的依赖，一般来只要有gcc、g++之类的编译必备工具就OK了，唯一一个比较特殊的依赖是Google家的snappy，因为WT引擎默认的表数据压缩方式是snappy(而不是zlib)。

```
wget http://source.wiredtiger.com/releases/wiredtiger-2.7.0.tar.bz2
tar xvf wiredtiger-2.7.0.tar.bz2 && cd wiredtiger-2.7.0
sudo apt-get install libsnappy-dev build-essential
./configure --enable-snappy
make
```

## 1.2 要恢复的文件

准备好要恢复的`collection*****.wt`，以及读取它必备的`_mdb_catalog.wt`、`sizeStorer.wt`、`storage.bson`、`WiredTiger`、`WiredTiger.basecfg`、`WiredTiger.lock`、`WiredTiger.turtle`、`WiredTiger.wt`。我们可以把这些文件放到一个新目录，比如本例放到`mongo_bak`下，目录结构如下：

```
collection********.wt
_mdb_catalog.wt
sizeStorer.wt
storage.bson
WiredTiger
WiredTiger.basecfg
WiredTiger.lock
WiredTiger.turtle
WiredTiger.wt
```

# 2.开始干吧

## 2.1 『打捞』出可以被恢复的部分

```
./wt -v -h ../mongo-bak -C "extensions=[./ext/compressors/snappy/.libs/libwiredtiger_snappy.so]" -R salvage collection******.wt
```

这一步操作会读取我们指定的`collection*****.wt`，忽略所有无法被恢复的数据，然后把新数据覆盖回去。当然你也可以修改参数让它把salvage后的文件写到另一个地方。

运行上述命令会输出`WT_SESSION.salvage 639400`这样的结果，后面那个数量其实就是所有能被恢复的数据。但是你现在还不能把这个直接读取到MongoDB。

## 2.2 做些必要的数据格式调整

因为上一步产生的wt文件还没法直接用MongoDB读，所以我们接下来这几步就利用wt的dump和load工具想办法把他们导入到MongoDB。

### 2.2.1 wt --> dump

```
./wt -v -h ../mongo-bak -C "extensions=[./ext/compressors/snappy/.libs/libwiredtiger_snappy.so]" -R dump -f ../collection.dump collection******
```

这一步会把我们刚打捞出来的健康的wt文件dump到上级目录的`collection.dump`文件（当然这个文件叫什么名、存哪里你自己定，下一步你还能找到就行）。注意这一步操作指定的collection不需要写wt扩展名了，程序很贴心有木有，省下3次敲键盘的体力可以干好多事情呢……我知道你都是复制粘贴的……

还要注意的是这一步程序是没有任何状态输出的（如果你看到了估计肯定是错误提示……），如果想看到进度的话可以用shell中的`ls -l`等命令通过观察`collection.dump`文件的变化来揣测进度，以及带给你一些程序确实运行成功了的安全感。

### 2.2.2 a new collection

这一步主要是为接下来的load做准备：我们要建立一个新的数据库，然后把上一步dump的数据导入进来，然后还有几个关键而鬼畜的步骤，后面都会提到，所以这一步还是老老实实跟着做一下吧。

来，我们先启动一个新的Mongo实例，我举个栗子，可以这么做：

```
mongod --dbpath tmp-mongo --storageEngine wiredTiger --nojournal
```

然后我们要连接这个实例并创建一个新集合

```
mongo
> use Recovery
> db.brokedCollection.insert({test: 1})
> db.brokedCollection.remove({})
> db.brokedCollection.stats()
```

我们创建了一个新的叫做Recovery的数据库，并且插入移除过文档，所以这些集合的数据文件会被生成。使用stats()方法可以查看集合所对应的wt文件名称，当然了，因为我们只使用了一个集合，所以跑到`tmp-mongo`目录下`ls`一下也就知道这个collecion对应的wt文件是哪个了……为什么要知道这个？当然是下一步要用了……

### 2.2.3 dump --> new wt

接下来是见证奇迹的时刻：我们的数据很快就可以重现Mongo了！

```
./wt -v -h ../tmp-mongo -C "extensions=[./ext/compressors/snappy/.libs/libwiredtiger_snappy.so]" -R load -f ../collection.dump -r collection******
```

这一步就是把前面转出来的dump文件读入上一步生成的collection文件，所以-h指定的当然是我们上一步用的新mongo实例的路径。执行这一步的时候需要先把Mongo关上，不然mongo进程会霸占着这个wt文件不让你操作。

这个操作是有一个进度展示的`table:collection-******: 1386220`

来来来，见证一下奇迹：现在可以再用2.2.2里的方式启动这个Mongo实例了

```
mongo
> show dbs   			//应该可以看到Recovery有数据了
> use Recovery
> show collections 		//应该可以看到brokedCollection里有数据了
> db.brokedCollection.count()   //是0？呐尼？
```

看到0的那一刻估计你又开始紧张了，别急我们慢慢来，如果到这一步就能解决问题的话我们何苦要折腾出2.2.2这一步。

```
> db.brokedCollection.find({}, {_id: 1})
```

接着执行这一条能看到数据，所以我估计你会再次燃起希望，接下来我们继续，让奇迹2出现吧：

## 2.3 完善一下

需要注意为了防止出错执行下面的步骤要确保MongoDB版本不小于3.2（因为3.2版本才是基于WiredTiger2.7构建的）。

```
mongodump
```
```
mongorestore --drop
```

没错就是这么简单，接下来我们就可以验证奇迹了。

```
mongo
> show dbs
> use Recovery
> show collections
> db.brokedCollection.count()
```

这次都回来了吧，一切正常了吧……接下来可以restore到任何你想要restore的地方了，如何使用可自由发挥。

# 3 收工

收工吧，累死了。

也许你已经受到上面步骤的启发想到更多有意思的恢复方法了，然而人生苦短，适可而止……

# MongoDB使用建议

对重要数据库所在机器操作前一定要提前backup一下，把万万没想到的损失降到最低；还有，除非真的不care数据的高可用性，不要随便关journal；单台机器上有多个库的话最好在配置文件中设置下directoryPerDB:true，让每个库有一个单独文件夹；有富余机器的话尽量做一下复制集……
