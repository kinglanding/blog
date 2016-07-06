---
layout: post
title: "我对KMP算法的理解"
date: 2014-05-19 22:56
comments: true
tags: 
- 数据结构
- 动态规划
- KMP
---


和朴素的字符串匹配算法相比，KMP加快匹配的关键在于当某个位置不匹配了，不在模版不再是向右滑动一个位置，而是跟匹配处相关的一个值。这样，在字符串的匹配时间就是和搜索串长度相关的一个值$$O(n)$$。

至于相关的资料，参阅了[六之再续：KMP算法之总结篇（12.09修订，必懂KMP](http://blog.csdn.net/v_july_v/article/details/7041827)，只是这个个人感觉有点凌乱。

定义要搜索的字符串模式是$$P$$,搜索的源是文本$$T$$.那让我们看看next数组怎么诞生的。

#### 局部匹配表

理解KMP算法的关键就是**局部匹配表**,或者形象的称作为**next数组**，
[算法导论](http://book.douban.com/subject/1885170/)p571页详细的描述了推导的过程，并给出了证明。但是仍然还是晦涩难懂的。

对于这个推论：设$$P$$是长度为$$m$$的模式，next是P的前缀函数，对于$$q=2,3,...,m$$

$$
next\left[ q \right] =\begin{Bmatrix} 0 & \quad if\quad { E }_{ q }\quad =\quad \phi  \\ 1+max\{ k\in { E }_{ q-1 }\}  & \quad if\quad { E }_{ q-1 }\quad =\phi  \end{Bmatrix}
$$

直观来说就是才用动规的方法，求$$next\left[ q \right]$$的最长前缀的长度。

那么好吧，再来个直观点的手写算法吧╮(╯▽╰)╭。

就以书上的例子为准，见下图。

![a)](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-05-19-my-understanding-about-kmp/1.png?raw=true "a)")


前q=5个字符匹配，标记为绿色。

![b)](https://github.com/aluenkinglee/aluenkinglee.github.io/blob/source/source/images/2014-05-19-my-understanding-about-kmp/2.png?raw=true "b)")

很明显，s+1的位置是无效的，但是s+2的位置很有可能是有效的，所以，可以直接向右移动2个位置。

推导中的使用的有用信息可以通过模式自身的比较来预处理计算得到。在这里，可以发现P的最长前缀同时也是$$P_5$$的一个真后缀$$P_3$$.这些信息可以预先急死俺，用数组next来表示，next[5]=3. 一般化的公式为，在位移s处有q个字符成功匹配，囊而下个有可能有效的位移是$$s=s-(q-next[q])$$.

上面提到了**真后缀**,让我们给出定义来。

**真前缀**： 字符串中去除1个至多个尾部字符的字符串集合。比如aabac的真前缀为a,aa,aab,aaba.

**真后缀**： 字符串中去除1个至多个头部字符的字符串集合。比如aabac的真后缀为c,ac,bac,abac.

好了 ，有了上面的定义，我们就可以简单，最重要的是明白计算next值了：`某个位置的next值为，到此位置的字符串产生的真前缀和真后缀`**并集**`中的最长元素的长度`。

就比如上面的那个例子，模式串为"ababaca"

```text
"a"的真前缀和真后缀都为空集，next[0]=0;

"ab"的真前缀为[b],真后缀为[a], next[1]=0;

"aba"的真前缀为[a,ab],真后缀为[a,ba],next[2]=1;

"abab"的真前缀为[a,ab,aba],真后缀为[b,ab,bab],next[3]=2;

"ababa"的真前缀为[a,ab,aba,abab],真后缀为[baba,aba,ba,a],next[4]=3;

"ababac"的真前缀为[a,ab,aba,abab,ababa],真后缀为[babac,abac,bac,ac,c],next[5]=0;

"ababaca"的真前缀为[a,ab,aba,abab,ababa,ababac],真后缀为[babaca,abaca,baca,aca,ca,a],next[5]=1;
```

#### 如何使用

见上面的例子吧，很好理解不是。

对了一下是代码。

```cpp
void compute_prefix(const char* p, int *next)
{
    next[0]=0;
    int len = strlen(p);
    int k =0,q;
    for(q=1; q < len; q++)
    {
        ///前q个字符中，前缀字符串集和后缀字符串集中最长的交集元素的长度
        while( (k > 0) && (p[k] != p[q]) )
            k = next[k-1];
        /// 请看前面的那个状态转移函数的公式
        if( p[k] == p[q] )
            k=k+1;
        next[q] = k;
    }
}
```

```cpp
void kmp_match(const char* T, const char* P)
{
    bool flag=false;
    int n=strlen(T);
    int m=strlen(P);
    int *next = new int[m];
    compute_prefix(P, next);
    int q=0;
    for (int i=0; i<n; i++)
    {
        while(q>0 && P[q]!=T[i])
            q=next[q-1];
        if(P[q]==T[i])
            q=q+1;
        if(q==m)
        {
            flag=true;
            cout<<"math occurs with "<<i-m+1<<endl;
            q=next[q-1];
        }
    }
    if(flag==false)
        cout<<"there is no math!"<<endl;
}
```

#### 总结

相对于朴素的匹配算法，这里并不是在匹配失败后直接无脑的外后移一位就可以了，而是利用了模式串本身的信息，**位移的距离=已经匹配的长度-该位置next值**。

希望对你有用。当然要是觉得有问题直接留言好了，一起学吧。