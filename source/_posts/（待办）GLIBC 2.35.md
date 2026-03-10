---
title: （待办）GLIBC 2.35
date: '2025-09-28 10:36:00'
updated: '2025-12-19 21:01:29'
---
## fastbin_dup
很简单，先把 tcachebin 填满，然后 free a -> b -> a 即可

## fastbin_dup_consolidate
1. 也是先填满 tcachebin
2. 然后 free 一个 fastchunk a
3. malloc 一个 largechunk 让 fastbin 里唯一的 chunk 与 topchunk 合并后被申请出来
4. 再 free 一次 fastchunk a

## fastbin_dup_into_stack
1. 实现 fastbin doublefree 之后
2. 在栈上伪造一个 fakechunk
3. 修改 doublefree 的 victimchunk 的 fd 为 fakechunk 
4. 清空 tcachebin 后申请或 calloc 绕过 tcachebin 申请出 fakechunk

fd 加密：

```python
*fd = (addr >> 12) ^ ptr;
# addr为指针本来的值
# ptr为存着这个值的地址
```

## fastbin_reverse_into_tcache
1. 先填满 tcachebin
2. 伪造 fakechunk a
3. 把 victim 先 free 进 fastbin，然后再填满 fastbin（如果 fakechunk 的 fd 是空的就只需要free victim 一个 chunk）
4. 将 victim 的 fd 改成 fakechunk a（记得加密）
5. malloc 7 次将 tcachebin 清空
6. 再 malloc 一次让 fastbin 当中的 chunk 流入 tcachebin
7. 再 malloc 出来的 chunk 就是 fakechunk a 了

## house of botcake
1. malloc 7 个 tcachechunk
2. 再 malloc prev、victim 两个 chunk 和一个 gapchunk
3. 先 free 掉前面 7 个 chunk 填满 tcachebin，
4. 然后再 free 掉 victim、prev（合并了）
5. malloc 1 个 tcachechunk 让 tcachebin 空出一个位置
6. 再 free 一遍 victim（进入了 tcachebin）
7. malloc 一个大 chunk（包括 prev、victim 和 victim 的 size 域）
8. 然后就可以控制到 victim 的 fd 指针（别忘了加密`(pos >> 12) ^ target_addr`）
9. 然后再 malloc 两次就可以获取到要读写的地址了

## mmap_overlapping_chunks
mmap 的堆块布置，可能位于 libc 地址之上或之下，具体分布跟内核版本相关

![](/images/fed049eed6ea7884e510a3463a131b84.png)

这里是在 libc 的低地址，这时候 mmap 是往低地址取

1. 先申请三个 mmapchunk1、2、3，chunk3 在最低地址
2. 然后修改 chunk3 的 size，能够覆盖到 chunk2 或 chunk1
3. 然后 free 掉 chunk3
4. 接着申请一个大于 chunk3 修改后的 size 的 mmapchunk
5. 这时候就把 chunk3 修改 size 后覆盖的 大 chunk 申请出来和低地址合并后获得申请的 chunk

