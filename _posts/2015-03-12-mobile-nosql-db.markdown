---
layout: post
title:  "浅谈 NoSql for Android platform"
categories: android,nosql,realm,couchbase
author: "killnono"
---


###支持资料
[Android Arsenal 站点关于几个Nosql的介绍.](https://android-arsenal.com/tag/155)

+ Realm.
+ SnappyDB.
+ SimpleNoSql.
+ couchbase


###Why Use Nosql on Moblie Platform
说到在我们的mobile应用中使用Nosql需求: 
 
1. **第一是需要做数据缓存**.
2. **第二点是server使用的MongoDB**.
3. **第三点其实和第一点有关联，因为服务端使用的是Document(文档)模型数据存储，请求拉回到移动端时,若使用同样的Nosql数据库格式，就基本无需改动直接存储**  

为了最少的对本地缓存数据做数据库设计和建模，就是使用和服务端类似的Nosql来存储数据。
其实本来是想使用sharepreference或是文本来做轻量级的缓存。但是考虑到在数据更新和查询时的效率（读一条文档需要read整个文本或是记录集等).

综合的来说:

1. 保持客户端和服务端数据存储模型的统一性,无需在设计关系型数据库模型(便洁)。
2. 相对来说直接文本和偏好存储，毕竟专业的nosql数据库具有更好更效率的数据操作。 

###关于目前可用的几个Nosql数据的概述
#####**Reaml**
----
 Realm ,支持Android和IOS+OSX，Realm for Android 2014.9.30,Realm相对来来说可能算是比较新的一个数据库框架。以Object或是Array形式对数据序列化存储，因为其早期开发的是IOS版本针对coredata存储的改进，所以基本概念中又很多ORM的感觉。  
 根据[Realm的官方文档](http://realm.io/docs/java/0.78.0/)教程实践时一般会出现几个问题。  
 
 ````
Caused by: io.realm.exceptions.RealmException: Could not find the generated proxy  class: Annotation processor may not have been executed.  
 ````
 
>官方FAQ:  
What does the exception ‘Annotation processor may not have been executed.’ mean?  
During compilation of your app, the model classes are processed and proxy classes are generated. If this annotation process fails, the proxy classes or their methods cannot be found. When your app begins running, it Realm will throw exception with message ‘Annotation processor may not have been executed.’. You may see this in case you use Java 6 as it doesn’t support inheriting annotations. You then have to add @RealmClass before your models. Otherwise it can often be solved by removing/cleaning all generated or intermediate files and rebuild.  
简单的说就是jdk<7的情况下会出现该问题，需要在realmObject前添加注解@RealmClass.

接下来可能会出现的问题是

````
io.realm.exceptions.RealmMigrationNeededException: Field 'name' not found for type 'YourBean' 
````
卸载下app清除旧的realmdb，重新运行就ok了。

Reaml的Android版本出的时间不久,他更像定义了一个Model的持久化框架,从对需要持久化的对象需要集成reaml的类，规范和注解的加入。对于稳定性来说,通用功能和常规的数据存储上基本没有问题.
官网的有很好的支持文档以及github的star和维护活跃度都是很高。

Tips:  
Realm和Gson配合使用导致Gson resolve StackOverflowError；
官方解决  

````
Gson gson = new GsonBuilder()
                .setExclusionStrategies(new ExclusionStrategy() {

                    @Override
                    public boolean shouldSkipField(FieldAttributes f) {
                        return f.getDeclaringClass().equals(RealmObject.class);
                    }

                    @Override
                    public boolean shouldSkipClass(Class<?> clazz) {
                        return false;
                    }
                })
                .create();
        String json = "{ name : 'John', email : 'john@corporation.com' }";
        User user = gson.fromJson(json, User.class);
````

但是对于比较复杂的数据存储，比如需要支持对象里面包涵对象，该类也需要是先realmobject;比如list类型的需要是使用reaml提供的reamllist容器。

#####**SnappyDB** 
----
简单粗暴的kv存储模式，可以直接存储一个对象（其机制会帮助你序列化）.假设我有一个同类型的对象数据集，
那么只能以array形式存储；查询的时候目前只能根据key值来读取，假设我只需要读取array集合里的某个对象，
那么会再取出整个array集合，然后再自己检索（目前从提供的简单的API来看只支持key查询,有兴趣的可以看看具体代码），个人觉得，是sharepreference的一个加强版，从他测试的数据图来说，效率还可以,使用也很简单。

#####**couchbase**
----
从mobile平台的nosql数据库来看，couchbase算比较提及的比较多。  
支持android，java，ios等多个平台和语言,基于文档形式的存储。
但是,其存储接口提供还是需要转为map才能存储，不支持直接JSON存储,虽然它提到最后是已JSON格式存储，
但是暴露在API接口上接受的是map。
文档默认都会生成一个\_id,\_rev字段,这在做缓存时会导致和我们本身的一些数据中的\_id冲突(目前来看是\_id被它自动覆盖了,并且只会发生在文档的最外层的\_id)。
简单的解决方案是，在需要设置自己的_id时，在创建Document时使用 
 
````
Document document = database.getDocument("myid2123");
````

基于自定义的\_id来获取或创建一个Document,也就可以实现默认\_id带来的问题，
当让前提是我们设计定义\_id是需要有唯一性的。

couchbase基本使用步骤:  

+ step1:根据上下文获取一个Manager

````
manager = new Manager(new AndroidContext(mContext), Manager.DEFAULT_OPTIONS);
````  

+ step2:通过指定的数据库名获取指定数据库  

````
this.db = manager.getDatabase("my-database");
````  

+ step3:创建一条记录写入数据库

````  
Map<String, Object> properties = new HashMap<String, Object>();
properties.put("type", "list");
properties.put("title", title);
properties.put("created_at", currentTimeString);
properties.put("owner", "profile:" + userId);
properties.put("members", new
ArrayList<String>());
Document document = database.createDocument();
document.putProperties(properties);
 ````  

具体的深度则可以考虑如何处理并发,锁机制,索引，视图等概念。

#####**SimpleNoSql**
----
略，直接看[其github和文档](https://github.com/Jearil/SimpleNoSQL)

 