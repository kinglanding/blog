---
layout: post
title: "Python处理带有时区的的字符串"
date: 2013-04-16 15:54
comments: true
tags: 
- python 
- 时间格式
---

最近在处理一些微博的数据，其中从服务器返回来的json串最后转换成了python中的字典，
只是可惜时间也被变成了字符串格式，好在python文档足够全且简单，可以使用datetime
中的strptime来解决，尽管如此还是在时区那卡了会

微博返回的时间数据格式如下：

##### “Fri Aug 12 14:09:31 +0800 2011″

然后我这样处理的

```
time.strptime('Fri Aug 12 14:09:31 +0800 2011', '%a %b %d %H:%M:%S %z %Y')
```
<!--more-->
当然也可以这样，只是试验下而已

```
dt2 = datetime.strptime('Fri Aug 12 14:09:31 +0800 2011', '%a %b %d %H:%M:%S %Z %Y')
```

然而却出现了如下的错误：

```
ValueError: time data 'Fri Aug 12 14:09:31 +0800 2011' does not match format '%a %b %d %H:%M:%S %Z %Y'
```

查了下[资料](http://stackoverflow.com/questions/10540399/strftime-does-not-return-abbreviated-time-zone)
发现这个跟系统有关系，而且这个是bug，（虽然开发者不承认，但是我觉得还有有点关系….虽然它又跟locals有关系。。。识别起来
确实很繁琐）比如现在我是在windows上处理的结果就是

```
>>> print time.strftime("%a, %d %b %Y %I:%M:%S %p %Z", time.localtime(10.5))
Thu, 01 Jan 1970 08:00:10 AM 中国标准时间
>>> print time.strftime("%a, %d %b %Y %I:%M %p %Z", time.gmtime())
Tue, 16 Apr 2013 08:33 AM 中国标准时间
>>>
```

和那个+8000格格不入，好吧 反正数据的处理设计不到时区，那么可以这样子做

```
>>> dt2 = datetime.strptime('Fri Aug 12 14:09:31 +0800 2011', '%a %b %d %H:%M:%S +0800 %Y')
>>> print dt2
2011-08-12 14:09:31
```

对现在的需求来说,反而更好.

### 参考

1. [issue6641](http://bugs.python.org/issue6641)

2. [python-strftime-gmtime-not-respecting-timezone](http://stackoverflow.com/questions/4788533/python-strftime-gmtime-not-respecting-timezone)

3. [strftime-does-not-return-abbreviated-time-zone](http://stackoverflow.com/questions/10540399/strftime-does-not-return-abbreviated-time-zone)

4. [微博使用](http://forum.open.weibo.com/read.php?tid=11780)

5. [strftime-and-strptime-behavior](http://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior)

6. [converting-string-into-datetime](http://stackoverflow.com/questions/466345/converting-string-into-datetime)