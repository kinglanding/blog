---
layout: post
title: "理解TCP中的序列号和确认号"
date: 2014-02-26 22:34
comments: true
categories: TCP libpcap wireshark 三次握手 C 四次握手
---

之所以写这么一篇文章是因为被wireshark的序列号搞晕了，我不知道你们是否这样，当你读这篇article时，肯定你已经熟悉了TCP那个三次握手，
或者是SYN,SYN|ACK,ACK. 而我要做的也就是这个，提取一个流的前N个packets，针对目前的需求，只需要在传输层截取UDP和TCP的前多少个流即可。
所以我得分析网络层的这些协议底层到底是怎么回事，在结合libpcap完成流的提取任务。

相对来说TCP还是个复杂的协议，而且值得清楚的认识以下。那么结合wireshark和代码来认识下TCP里面的东西。

<!--more-->

[这里](https://github.com/aluenkinglee/stuff/blob/master/cplusplus/traffic_parser/b.pcap?raw=true 'pcap文件')是个已经准备好的pcap文件，本文结合这个文件对此进行描述。

这个文件分析的是微信在单一环境中的网络行为，所以比较简单。打开这个文件，找到开始的几个右键单击选择`Follow TCP Stream`，或者在filter那里输入`tcp.stream eq 1`可以看到这里。

![Follow TCP Stream](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-02-26-li-jie-tcpzhong-de-xu-lie-hao-he-que-ren-hao/tcp1.png?raw=true "Follow TCP Stream")


####三次握手

TCP利用了8个标志位，在头部位置，以此来控制链接的状态，对我们最有用的就是SYN，ACK，FIN了。

* SYN - (Synchronize) Initiates a connection
* FIN - (Final) Cleanly terminates a connection
* ACK - Acknowledges received data

下面将会看到，一个packet包含了多个flag set。

######对于这个流的第一个，注意Flags里面出了SYN位是1之外，其它都是0。

![第一次握手](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-02-26-li-jie-tcpzhong-de-xu-lie-hao-he-que-ren-hao/tcp2.png?raw=true "第一次握手")


######接下来在看第二个，注意它有两个标志位被设置为1，分别是SYN和ACK。

![第二次握手](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-02-26-li-jie-tcpzhong-de-xu-lie-hao-he-que-ren-hao/tcp3.png?raw=true "第二次握手")

######对于第三个packet，就只有ACK设置了。

![第三次握手](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-02-26-li-jie-tcpzhong-de-xu-lie-hao-he-que-ren-hao/tcp4.png?raw=true "第三次握手")

这就是最初的TCP三次握手。

####序列号和应答号（SEQ和ACK）

假设客户端为A，服务器为B，双方都为护着一个32bit的序列号，用来追踪传输了多少数据。。这个号包括了之前所传输的负载的
大小，由另一端的应答号来应答“你传的数据我都收到了。。”

当一个机器开始初始TCP序列号时，它是随机的！！不然会出现序列号攻击。。（我忘记了，在用那个wireshark后成功被它的相对序列号迷惑了==）,是一个0～ $2^{32}-1$的数。相对序列号是给人看的，所以像wireshark之辈使用它是为了人们方便阅读理解的。
（选择不启用相对序列号：选择`Edit > Preferences... `不启用那个`Relative sequence numbers and window scaling `就可以了）

接下来让我们结合下图来看这个流的行为。在`Statistics > Flow Graph...`选择`TCP flow`。

![流图](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-02-26-li-jie-tcpzhong-de-xu-lie-hao-he-que-ren-hao/tcp5.png?raw=true "流图")

这张图很容易理解就不说了。使用这张流图可以很方便的帮助我们理解他们是怎么工作的。

######packet #1

A向B发了一个请求，我们可以在frame的tcp中分析出来，SYN=1而ACK=0，这意味这是一个流的起始包。这里使用的是相对序列号，所以为0.

######packet #2

B收到了响应，恩，因为这是会话的开始，所以B这边也生成了一个随机的序列号，只不过在这里也显示为0了。SYN置为1。另外ACK也置为1，表明收到了A的响应。
(注意！虽然A没有发送任何负载payload，B仍然把ACK置为1，是因为收到的SYN或者FIN触发了这个增1行为。这儿不会涉及到任何负载长度的计算，因为带有这样信号的包不会携带负载的。)

######packet #3

和#2一样，A回应了B的响应（SEQ=0，ACK=1）所以ACK为1.自己的SEQ因为收到的包中有SYN所以变为1。此时，双方的SEQ都是1，这种现象在所有TCP开始建立连接时候都是一样的。

######packet #4

A这个包此时带有负载，这里的SEQ是1，因为上个包（#3）没有传输任何数据，ACK也是1，因为A没有传输任何数据。注意！packet的长度是341，但是我们计算的是传输层的数据长度--负载的长度，所以是ACK — LEN = LEN（#4）-LEN（#3） = 341 - 66 = 275。

######packet #5

这个包是B（#5）对A（#4）发送数据的响应，此时B的ACK加上负载的长度是275变为276，表示我B收到了你A传输的数据payload（#4）. B此时的SEQ仍为1.

######packet #6

这个包是B对A放送的HTTP响应，因为之前它（B）所有的包都没有负载，所以SEQ仍然为1，而ACK还是276.负载长度为627.

######packet #7

好吧，这个例子有点特殊，到这里为止B的数据就发送完了。。。所以FIN置为1，表明你（A）要得我（B）都给完了，没我的事儿了，SEQ加上我发送的数据长度为628，你那边确认之后应该和我一样才对。ACK还是之前你给我发送的那些数据，还是这些276.

######packet #8

A:收到了所有的数据，我先确认给你我数据我收到了，ACK加上627变为628。我之前发送的数据截止到目前是276，没错，我们对上了。

######packet #9 

A:既然我都要到了，那么我们就分手吧~~ 同意，FIN置为1，因为从B收到了带有FIN的报文（#7），所以ACK+1，变为629。因为上个数据包#8 没有发送任何数据，所以这里的SEQ不变,

######packet #10

因为#9带有FIN，所以SEQ自增1.ACK不变。

关于代码，请看下篇。

## 参考

1. [TCP sequence number question](http://stackoverflow.com/questions/2672734/tcp-sequence-number-question)

2. [Transmission Control Protocol](http://en.wikipedia.org/wiki/Transmission_Control_Protocol)


