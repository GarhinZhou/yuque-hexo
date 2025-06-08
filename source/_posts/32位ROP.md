---
title: 32位ROP
date: '2025-06-06 15:29:44'
updated: '2025-06-06 16:00:54'
---
才发现之前一直没了解 32 位的 ROP 怎么写，遇到的题目都是硬动调调出来的，后面查了查才知道原来返回地址是跟在调用地址后面的，再往后才是调用函数的参数

## 例子
解释一下`p32(read_addr) + p32(system_addr) + p32(0) + p32(bss_addr) + p32(256)`为什么如此构造

read_addr 是函数执行结束后的返回地址，而这里的read_addr指向的是plt表，在执行plt表所指向的read_addr函数时，stack中的数展现如下：

| stack | 作用 |
| --- | --- |
| system_addr | read执行完成后的return address |
| p32(0) | read函数的第1个参数 unsigned int fd |
| p32(bss_addr) | read函数的第2个参数 char *buf |
| p32(255) | read函数的第3个参数 size_t count |


这样的栈结构相当于，执行了read(0, bss_addr, 255)后，重新跳转到system_addr，而跳转到system_addr后，stack如下：

| stack | 作用 |
| --- | --- |
| p32(0) | system函数的return address |
| p32(bss_addr) | system函数的第1个参数 char *buf |
| p32(255) | 用不到 |


也就是说，read结束后，程序走到了system的plt中去执行程序，而此时将bbs_addr当做参数传给了system，相当于执行了system(bss_addr)。

由于bss_addr已经被我们写入了/bin/sh，此时system函数调用时将会给我们返回一个shell，所以system的return address是什么并不重要

可以利用`pop`指令，实现对栈的清除，已知 read 函数用到了3个参数，可以利用三个 pop + ret 指令的 gadget 来作为返回地址从而跳过 read 函数的三个参数接着 ROP 下去

```python
0x080486b9: pop esi; pop edi; pop ebp; ret;
```

