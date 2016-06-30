---
layout: post
title: "Linux下通过端口杀死进程"
date: 2014-05-20 10:41
comments: true
tags: 
- linux
- 进程
- 常用命令
---

就是把一些常用命令记录下来拉。一直在用，可就是容易忘记命令。

floodlight 控制器启动之后，因为不正常的关闭程序（ctrl+c）造成6633端口还在被占用。

流程如下：

1. netstat -nlp 查看占用端口号的服务

2. 找到该端口号的进程，可以使用grep

3. 找到该进程id

4. kill pid

##### 查看占用端口号的服务

可以使用netstat命令

```text
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name  
tcp        0      0 0.0.0.0:6633            0.0.0.0:*               LISTEN      6267/java              
tcp        0      0 0.0.0.0:3307            0.0.0.0:*               LISTEN      12711/              
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      3936/httpd          
tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN      3910/              
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3753/sshd          
tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      3786/    
```
##### 用管道符给grep处理

```bash
 netstat -nlp | grep 6633
```


既然取出一行了，那就容易了，再筛选一下，用awk分割取出其中一个

##### 读取出端口号

```bash
netstat -nlp | grep 6633 | awk '{print $7}'
```

6267/java

意思是取第七个字段，这里默认应该是用tab字符分割的，已经读取出来了，但是还得拿到/前面的数字

##### 取/前面的数字就可以了，这里还是可以用awk处理

```bash
netstat -nlp | grep 6633 | awk '{print $7}' | awk -F"/" '{ print $1 }'
```

6267

##### 把这个数字传给kill就可以

kill命令不能跟在管道符后面继续处理了，会出错的.

```bash
kill ['] netstat -nlp | grep 6633 | awk '{print $7}' | awk -F"/" '{ print $1 }' [']
```

> 其它类似的命令

1. 查看端口属于哪个程序？端口被哪个进程占用

也可以使用lsof命令。

```bash
$ lsof -i :6633

COMMAND   PID   USER      FD   TYPE    DEVICE  SIZE/OFF NODE NAME
java    7838   kinglee   50r   IPv6  35452317       0t0  TCP *:6633 (LISTEN)
```

### reference

1. [netstat](http://baike.baidu.com/link?url=idG3yCyykQj00QVBW_jreekVZWjIGU5urL553dG9o4ZYgwpbnjd7jJ2DVjxrm5EZ)

2. [Linux下通过端口杀死进程](http://www.cnblogs.com/peter9/archive/2011/07/28/2362156.html)

3. [linux下杀死进程（kill）的N种方法](http://blog.csdn.net/andy572633/article/details/7211546)