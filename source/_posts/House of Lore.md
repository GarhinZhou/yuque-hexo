---
title: House of Lore
date: '2025-03-25 10:49:51'
updated: '2025-04-07 21:39:55'
---
## 条件
1. libc 版本：2.23 至今
2. 有堆溢出或 UAF，可以修改到 smallchunk 的 bk

## 原理
控制 smallbin 的 bk 指针，示例如下：

+ 申请 chunk A、chunk B、chunk C，其中 chunk B 大小位于 smallbin
+ 释放 B，申请更大的 chunk D，使得 B 进入 smallbin
+ 写 A，溢出修改 B 的 bk，指向地址 X，这里有 fake chunk
+ 布置 X->fd == &B
+ 分配两次后即可取出位于 X 地址处的 fake chunk

感觉跟 fastbin attack 很像，有这个利用条件应该也可以用 fastbin attack



如果在 malloc 的时候，申请的内存块在 small bin 范围内，那么执行的流程如下：

```python
/*
   If a small request, check regular bin.  Since these "smallbins"
   hold one size each, no searching within bins is necessary.
   (For a large request, we need to wait until unsorted chunks are
   processed to find best fit. But for small ones, fits are exact
   anyway, so we can check now, which is faster.)
 */

if (in_smallbin_range(nb)) {
	// 获取 small bin 的索引
	idx = smallbin_index(nb);
	// 获取对应 small bin 中的 chunk 指针
	bin = bin_at(av, idx);
	// 先执行 victim= last(bin)，获取 small bin 的最后一个 chunk
	// 如果 victim = bin ，那说明该 bin 为空。
	// 如果不相等，那么会有两种情况
	if ((victim = last(bin)) != bin) {
		// 第一种情况，small bin 还没有初始化。
		if (victim == 0) /* initialization check */
			// 执行初始化，将 fast bins 中的 chunk 进行合并
			malloc_consolidate(av);
		// 第二种情况，small bin 中存在空闲的 chunk
		else {
			// 获取 small bin 中倒数第二个 chunk 。
			bck = victim->bk;
			// 检查 bck->fd 是不是 victim，防止伪造
			if (__glibc_unlikely(bck->fd != victim)) {
				errstr = "malloc(): smallbin double linked list corrupted";
				goto errout;
			}
			// 设置 victim 对应的 inuse 位
			set_inuse_bit_at_offset(victim, nb);
			// 修改 small bin 链表，将 small bin 的最后一个 chunk 取出来
			bin->bk = bck;
			bck->fd = bin;
			// 如果不是 main_arena，设置对应的标志
			if (av != &main_arena) set_non_main_arena(victim);
			// 细致的检查
			check_malloced_chunk(av, victim, nb);
			// 将申请到的 chunk 转化为对应的 mem 状态
			void *p = chunk2mem(victim);
			// 如果设置了 perturb_type , 则将获取到的chunk初始化为 perturb_type ^ 0xff
			alloc_perturb(p, bytes);
			return p;
		}
	}
}
```

如果我们可以修改 small bin 的最后一个 chunk 的 bk 为我们指定内存地址的 fake chunk，并且同时满足之后的 `bck->fd != victim` 的检测，那么我们就可以使得 small bin 的 bk 恰好为我们构造的 fake chunk。也就是说，当下一次申请 small bin 的时候，我们就会分配到指定位置的 fake chunk

## 重点
要将一个 chunk 放进 smallbin 中，前提是它比 fastchunk 大，先把它释放进 unsortedbin 里面，然后再申请一个比它还要大的 chunk，在遍历一遍 unsortedbin 之后，才会把它放进 smallbin 里面，例如：

`free((void*)victim)`<font style="color:rgba(0, 0, 0, 0.87);">，victim 会被放入到 unsort bin 中去，然后下一次分配的大小如果比它大，那么将从 top chunk 上分配相应大小，而该 chunk 会被取下 link 到相应的 bin 中。如果比它小 (相等则直接返回)，则从该 chunk 上切除相应大小，并返回相应 chunk，剩下的成为 last reminder chunk , 还是存在 unsorted bin 中</font>

