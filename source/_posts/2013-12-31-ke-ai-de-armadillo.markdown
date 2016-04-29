---
layout: post
title: "可爱的Armadillo"
date: 2013-12-31 10:25
comments: true
categories: Armadillo openblas lapack armadillo链接错误 C++ Machine Learning
---

最近又重新看了一下coursera上的[机器学习](https://class.coursera.org/ml-004/lecture/index)(Andrew Ng讲的)，比起之前上老师的课和[机器学习](http://book.douban.com/subject/1102235/ '机器学习')这本书来说，简直好太多。当初选修这个课程的时候明显感到编程实践环节太少，很不适应，只有一个大作业而已。好在这里的Quara和Programming Excises很给力。

我的设想是这样，octave作为一种快速验证想法的工具不适合应用在实际的生产环境中的，毕竟计算速度还是可以依靠集群和并行化来加快大数据处理。在此参考了C++的[线性代数库](http://en.wikipedia.org/wiki/Comparison_of_linear_algebra_libraries)之后选择了Armadillo,毕竟经常更新并且从官网资料来看，和octave代码相似便于移植，再说，从它本身和其他的对比来看，速度也是相当快的。

不过，在安装完Armadillo之后，编译example目录下的例子并不通过，提示

<!--more-->


```
/tmp/cc9ckDKG.o: In function `void arma::gemv<false, false, false>::apply_blas_type<double>(double*, arma::Mat<double> const&, double const*, double, double)':
example1.cpp:(.text._ZN4arma4gemvILb0ELb0ELb0EE15apply_blas_typeIdEEvPT_RKNS_3MatIS3_EEPKS3_S3_S3_[_ZN4arma4gemvILb0ELb0ELb0EE15apply_blas_typeIdEEvPT_RKNS_3MatIS3_EEPKS3_S3_S3_]+0x7a): undefined reference to `wrapper_dgemv_'
/tmp/cc9ckDKG.o: In function `main':
example1.cpp:(.text.startup+0x1a5d): undefined reference to `wrapper_ddot_'
collect2: error: ld returned 1 exit status
```

然后看了readme里面的编译链接部分，发现即使尝试

```
g++ example1.cpp -o example1 -O2 -llapack -lblas
```

也是不行。所以不甘心的看了`/usr/include/armadillo_bits/config.hpp`，找到了`#define ARMA_USE_WRAPPER`,并把它注释掉,就像这样

```
//// #define ARMA_USE_WRAPPER
//// Comment out the above line if you're getting linking errors when compiling your programs,
//// or if you prefer to directly link with LAPACK and/or BLAS.
//// You will then need to link your programs directly with -llapack -lblas instead of -larmadillo
```

在用上面的命令就可以了。

至于怎么安装，还是请看Readme吧各位。希望这个帖子能帮助有类似问题的人。（在debain系列的linux上有可能会有这样的问题。）

接下来，应该就会把之前看过的视频和资料的东西在整理一下，并借用这个库实现应该实现的部分。
