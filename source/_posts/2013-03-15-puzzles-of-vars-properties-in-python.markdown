---
layout: post
title: "python中关于变量属性的迷惑"
date: 2013-03-15 17:06
comments: true
categories: python
---

当初使用C++，java的时候，变量的属性（可访问性，scope这类的意思）如public，private，protected，static这类很好理解。

但是Python中默认的成员函数，成员变量都是公开的(public),而且python中没有类似public,private等关键词来修饰成员函数，成员变量。

<!-- more -->

在python中定义私有变量只需要在变量名或函数名前加上 ”__“两个下划线，那么这个函数或变量就会为私有的了。对static这样的很是好奇。

于是便有了这样子的实验。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
class A:
    value = 0

o1 = A()
o2 = A()

print A,o1,o2
print A.value,o1.value,o2.value

o1.value += 10
print A.value,o1.value,o2.value

A.value += 5
print A.value,o1.value,o2.value

o2.value += 5
print A.value,o1.value,o2.value
```

这是我机器上的测试结果，A没有显示地址??怎么回事，两个实例都有自己的地址:

```
__main__.A <__main__.A instance at 0x01262AD0> <__main__.A instance at 0x013091C0>
0 0 0
0 10 0
5 10 5
5 10 10
```

前两行的结果都很好理解，但是第三行，之前o2的值一直没有改变，o1的值改变，在A的值改变之后o2也随之改变了，这个时候….看上去，o2还是在引用A的值，但是在o2自己的值改变之后就不在去引用类的值了。

#####由此可见：

* 由类派生出来的实例在未操作之前，都是简单的引用类的那些值

* 公共属性的值有点类似于大家C++类中的那种静态变量啊

* 只要实例稍作改变，就不会在引用类了。