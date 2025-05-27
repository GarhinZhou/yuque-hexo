---
title: House of Pig
date: '2025-05-26 19:22:32'
updated: '2025-05-26 19:28:29'
---
## 利用思路
好长...复现难度好大...

1. 先用 UAF 漏洞泄露 libc 地址 和 heap 地址
2. 再用 UAF 修改 largebin 内 chunk 的 fd_nextsize 和 bk_nextsize 位置，完成一次 largebin attack，将一个堆地址写到 __free_hook-0x8 的位置，使得满足之后的 tcache stashing unlink attack 需要目标 fake chunk 的 bk 位置内地址可写的条件
3. 先构造同一大小的 5个 tcache，继续用 UAF 修改该大小的 smallbin 内 chunk 的 fd 和 bk 位置，完成一次 tcache stashing unlink attack。由于前一步已经将一个可写的堆地址，写到了__free_hook-0x8，所以可以将 __free_hook-0x10 的位置当作一个 fake chunk，放入到 tcache 链表的头部。但是由于没有 malloc 函数，我们无法将他申请出来
4. 最后再用UAF 修改 largebin 内 chunk 的 fd_nextsize 和 bk_nextsize 位置，完成第二次 largebin attack，将一个堆地址写到 _IO_list_all 的位置，从而在程序退出前 flush 所有 IO 流的时候，将该堆地址当作一个 FILE 结构体，我们就能在该堆地址的位置来构造任意 FILE结构了
5. 在该堆地址构造 FILE 结构的时候，重点是将其 vtable 由 _IO_file_jumps 修改为 _IO_str_jumps，那么当原本应该调用 IO_file_overflow 的时候，就会转而调用如下的 IO_str_overflow。而该函数是以传入的 FILE 地址本身为参数的，同时其中会连续调用 malloc、memcpy、free 函数（如下图），且三个函数的参数又都可以被该 FILE 结构中的数据控制。那么适当的构造 FILE 结构中的数据，就可以实现利用 IO_str_overflow 函数中的 malloc 申请出那个已经被放入到 tcache 链表的头部的包含 __free_hook 的 fake chunk；紧接着可以将提前在堆上布置好的数据，通过 IO_str_overflow 函数中的memcpy 写入到刚刚申请出来的包含__free_hook的这个 chunk，从而能任意控制 __free_hook ，这里可以将其修改为 system函数地址；最后调用 IO_str_overflow 函数中的 free 时，就能够触发 __free_hook ，同时还能在提前布置堆上数据的时候，使其以字符串 “/bin/sh\x00” 开头，那么最终就会执行 system(“/bin/sh”)

