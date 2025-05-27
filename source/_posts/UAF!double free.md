---
title: UAF/double free
date: '2025-02-19 22:14:53'
updated: '2025-04-07 21:39:55'
---
## UAF
using after free 效果：任意地址写

有UAF意味着能掌控一切，能写fd的话，就能申请到任意位置

## double free
漏洞来自于ptmalloc2堆管理器本身 2.23fastbin c1->c2->c1   效果：造成UAF

**现在在glibc不同的环境下，有着**[**不同的double free手法**](https://blog.csdn.net/chennbnbnb/article/details/109284780)

+ 在glibc2.27之前，主要是fastbin double free：

fastbin在free时只会检查现在释放的chunk，是不是开头的chunk，因此可以通过free(C1), free(C2), free(C1)的手法绕过，并且会在fastbin取出时，检查size字段是不是属于这个fastbin，因此往往需要伪造一个size

跟着自己捋了一遍 doublefree 的具体过程（是 fastbin 的，glibc2.27~glibc2.28 的 tcache double free 相对简单一点还好理解）

![](/images/551083eee04c2e57efb6a01f444eb062.jpeg)

+ glibc2.27~glibc2.28，主要是tcache double free：

相较于fastbin double free，tcache完全没有任何检查，只需要free(C1), free(C1)就可以构造一个环出来

+ glibc2.29～glibc2.31，tcache加入了检查机制

