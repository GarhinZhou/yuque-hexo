---
title: CTF Show刷题
date: '2025-06-14 16:02:37'
updated: '2025-06-14 17:19:11'
---
## 31
除 canary 都开了，有栈溢出

got 表是 Full relro 的，这说明 got 表不可写，而且在 got 表在程序执行一开始就已经解析了

![](/images/bba35fc4cac4bf47c2e5bf0c32688eb9.png)

但是这里多了一句`mov ebx, [ebp+var_4]`

而程序调用 puts 的时候是从 ebx 来获取 got 表起始地址的，所以需要



