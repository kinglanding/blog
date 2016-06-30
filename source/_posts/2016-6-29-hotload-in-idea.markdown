---
title: 在idea中启用热加载
date: 2016-06-29 12:43:31
tags:
- hotload
- 热加载
- idea
---

idea真是开发神奇，越来越喜爱。修改个类或者资源文件啥的不在用重新部署下容器，
也不用像eclipse那样装上JRebel的插件，有点太重。只需要这样：

![use-hotload hotload](/img/post/use-hotload.png)

如此，就好，注意On frame deactivation这个选项，实际就是当你离开当前的frame时，
idea就默默为你编译工程文件，然后放到explored的war中去了。本质就是将改好后的类
编译一下就好。