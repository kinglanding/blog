---
layout: post
title: "MySql交换两列"
date: 2013-06-14 15:35
comments: true
tags: 
- python 
- mysql
---

事情的起因是这个发现数据库中出现了某两列数据正好颠倒了，大部分数据都是正常的，
这些错误的原因是数据源有问题，今天在进一步的做标签聚类的时候分析的时候发现这
些占很小比例的误差，辛亏发现了，否则做出来的结果有可能会郁闷很久。

```
select id,place_lat,place_lon from `nearbyinfo` where `nearbyinfo`.`place_lat`>0;
```

正常的数据都是< `39.xxx`, `160.2xxx` >之间，如图所示:
<!--more-->

![正常数据](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2013-06-14-mysqljiao-huan-liang-lie/11.png?raw=true "正常数据")

但是幸好及时发现了存在这样的数据，

![异常数据](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2013-06-14-mysqljiao-huan-liang-lie/12.png?raw=true "异常数据")

一开始想的是想写sql语句试着把这两栏的数据给交换过来，可是sql太渣…然后就想着找
到这些ID和他们的坐标值（都已经出来了），然后用python读出来写成update的语句，
然后在执行一下就可以了，这也太…


辛亏最终还是找到了，在这还是完整的说明一下吧。首先创建一个表，在Mysql中

```
CREATE DATABASE  IF NOT EXISTS `weibodata` 
USE `weibodata`;
DROP TABLE IF EXISTS `test`;
CREATE TABLE `test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a` varchar(45) DEFAULT NULL,
  `b` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
LOCK TABLES `test` WRITE;
INSERT INTO `test` VALUES (1,'-1','4'),(2,'3','2');
UNLOCK TABLES;
```

所以看起来会是这个样子:

![交换之前](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2013-06-14-mysqljiao-huan-liang-lie/12.png?raw=true "交换之前")

然后执行下面的语句，会发生第二行swap了

```
UPDATE test t1,test t2 SET t1.a=t1.b,t2.b=t2.a where t1.id =t2.id and t1.a>0;
```

![交换之后](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2013-06-14-mysqljiao-huan-liang-lie/12.png?raw=true "交换之后")

在尝试一下就会变成原样。

问题解决了。

```
select id,place_lat,place_lon from nearbyinfo where place_lat >100;
UPDATE nearbyinfo t1,nearbyinfo t2 SET t1.place_lat=t1.place_lon,t2.place_lon=t2.place_lat where t1.id = t2.id and t1.place_lat>100;
select id,place_lat,place_lon from nearbyinfo where id in (1025,4974,4814,2685);
```