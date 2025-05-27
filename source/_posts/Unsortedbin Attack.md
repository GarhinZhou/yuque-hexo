---
title: Unsortedbin Attack
date: '2025-03-20 14:34:52'
updated: '2025-04-19 21:03:34'
---
先回顾一下 Unsorted Bin 的基本来源以及基本使用情况

## 基本来源
1. 当一个较大的 chunk 被分割成两半后，如果剩下的部分大于 MINSIZE，就会被放到 unsorted bin 中
2. 释放一个不属于 fast bin 的 chunk，并且该 chunk 不和 top chunk 紧邻时，该 chunk 会被首先放到 unsorted bin 中。关于 top chunk 的解释，请参考下面的介绍
3. 当进行 malloc_consolidate 时，可能会把合并后的 chunk 放到 unsorted bin 中，如果不是和 top chunk 近邻的话

## 基本使用情况
1. Unsorted Bin 在使用的过程中，<font style="background-color:#FBDE28;">采用的遍历顺序是 FIFO，即插入的时候插入到 unsorted bin 的头部，取出的时候从链表尾获取</font>
2. 在程序 malloc 时，如果在 fastbin，small bin 中找不到对应大小的 chunk，就会尝试从 Unsorted Bin 中寻找 chunk。如果取出来的 chunk 大小刚好满足，就会直接返回给用户，否则就会把这些 chunk 分别插入到对应的 bin 中

## 泄露 libc 地址
Unsorted Bin 在管理时为循环双向链表，若 Unsorted Bin 中有两个 bin，那么该链表结构如下

![](/images/904d2e8b6ab7e76e63cfb788869306ae.png)

我们可以看到，在该链表中必有一个节点的 fd 指针会指向 main_arena 结构体内部

### 原理
如果我们可以把正确的 fd 指针 leak 出来，就可以获得一个与 main_arena 有固定偏移的地址，这个偏移可以通过调试得出。而main_arena 是一个 struct malloc_state 类型的全局变量，是 ptmalloc 管理主分配区的唯一实例。说到全局变量，立马可以想到他会被分配在 .data 或者 .bss 等段上，那么如果我们有进程所使用的 libc 的 .so 文件的话，我们就可以获得 main_arena 与 libc 基地址的偏移，实现对 ASLR 的绕过

那么如何取得 main_arena 与 libc 基址的偏移呢？有两种思路：

#### 通过 __malloc_trim 函数得出
在 malloc.c 中有这样一段代码

```python
int
__malloc_trim (size_t s)
{
  int result = 0;

  if (__malloc_initialized < 0)
    ptmalloc_init ();

  mstate ar_ptr = &main_arena;//<=here!
  do
    {
      __libc_lock_lock (ar_ptr->mutex);
      result |= mtrim (ar_ptr, s);
      __libc_lock_unlock (ar_ptr->mutex);

      ar_ptr = ar_ptr->next;
    }
  while (ar_ptr != &main_arena);

  return result;
}
```

注意到 mstate ar_ptr = &main_arena; 这里对 main_arena 进行了访问，所以我们就可以通过 IDA 等工具分析出偏移了。

![](/images/3dfc7757b80d32cbbb6c329768b61b7f.png)

比如把 .so 文件放到 IDA 中，找到 malloc_trim 函数，就可以获得偏移了

#### 通过 __malloc_hook 直接算出
比较巧合的是，main_arena 和 __malloc_hook 的地址差是 0x10，而大多数的 libc 都可以直接查出 __malloc_hook 的地址，这样可以大幅减小工作量。以 pwntools 为例

```python
main_arena_offset = ELF("libc.so.6").symbols["__malloc_hook"] + 0x10
```

这样就可以获得 main_arena 与基地址的偏移了

### 实现方法
一般来说，要实现 leak，需要有 UAF，将一个 chunk 放入 Unsorted Bin 中后再打出其 fd。一般的笔记管理题都会有 show 的功能，对处于链表尾的节点 show 就可以获得 libc 的基地址了

特别的，CTF 中的利用，堆往往是刚刚初始化的，所以 Unsorted Bin 一般都是干净的，当里面只存在一个 bin 的时候，该 bin 的 fd 和 bk 都会指向 main_arena 中

另外，如果我们无法做到访问链表尾，但是可以访问链表头，那么在 32 位的环境下，对链表头进行 printf 等往往可以把 fd 和 bk 一起输出出来，这个时候同样可以实现有效的 leak。然而在 64 位下，由于高地址往往为 \x00，很多输出函数会被截断，这个时候可能就难以实现有效 leak

## 进入检查机制
释放 chunk 进入 unsortedbin 的机制：

![](/images/6768ed524aa0e2604fae9b5ebbe8ece2.png)

> 这个机制在很多 libc 版本使用，没有 tcache 的时候也是
>

伪造连续三个chunk，后面两个用来绕过unsortedbin的检查，前面的用来利用编辑功能来篡改size，从而让伪造的 chunk被free进unsortedbin

