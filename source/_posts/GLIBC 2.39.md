---
title: GLIBC 2.39
date: '2025-09-05 11:41:56'
updated: '2025-10-04 23:31:02'
---
## safe-linking机制（从2.32版本启用）
将原本正常fd值与当前堆块用户地址（也就是fd的位置）右移12位后进行异或，

得到的值当作加密后的fd

这个时候第一个进入bin当中的chunk的fd就不是0了，而是堆基址右移12位的值

因为堆基址本来最低的一个半字节（12位）就是0，所以泄露出来左移12位可以直接获得堆的基址

劫持hook的时候就把`(fd的位置>>12)^(hook_addr)`替换fd就可以了

但是poc里面的decrypt函数没看懂key是哪里来的（标记这句没看懂）

```c
long decrypt(long cipher)
{
    puts("The decryption uses the fact that the first 12bit of the plaintext (the fwd pointer) is known,");
    puts("because of the 12bit sliding.");
    puts("And the key, the ASLR value, is the same with the leading bits of the plaintext (the fwd pointer)");
    long key = 0;
    long plain;

    for(int i=1; i<6; i++) {
        int bits = 64-12*i;
        if(bits < 0) bits = 0;
        plain = ((cipher ^ key) >> bits) << bits;
        key = plain >> 12;
        printf("round %d:\n", i);
        printf("key:    %#016lx\n", key);
        printf("plain:  %#016lx\n", plain);
        printf("cipher: %#016lx\n\n", cipher);
    }
    return plain;
}
```

## fastbin_dup
在这个版本的fastbin可以doublefree，只是条件比较严苛，看poc来说就是：

1. 先malloc申请8个chunk
2. free7个填满tcachebin
3. 然后用calloc绕过tcache申请3个chunk（a/b/c，c当gap）
4. 按照a-b-a的顺序实现doublefree

试了试直接malloc10个chunk，free7个填满tcache再按a-b-a来free也可以实现

利用doublefree改fd记得先泄露出堆地址再加密：`(addr >> 12) ^ ptr`

## fastbin_dup_consolidate
前提：有UAF

利用malloc_consolidate实现doublefree

1. 填满tcachebin
2. malloc一个chunk p1
3. 再free p1（这时候就和topchunk合并了）
4. malloc一个大于所有tcachechunk的chunk p2，这时候p1=p2
5. 再free一次p1实现doublefree
6. 这时候再malloc一个p2一样大小的chunk p3，这时候p2和p3都指向同一个被free了的chunk

## fastbin_revserse_into_tcache
前提：有heap地址、malloc大小属于fastbin

跟unsortedbin attack相似，可以向任意位置写入一个大的数值

1. malloc14个chunk
2. free7个填满tcachebin
3. free7个填满fastbin，其中第一个被free，也就是第一个进入fastbin的chunk是要改fd的victim，fd改成要填入大数值的地址减去0x10（如果要填入数值的地方本来数值是0，可以只free一个chunk也就是victim）
4. malloc7个chunk清空tachebin，这时候fastbin还是满的
5. 再malloc一个chunk，这时候因为fastbin里面的chunk会涌进tcachebin里面，所以第一个进入fastbin的victim就变成最后一个进tcachebin的chunk
6. 这个时候就让目标位置写上一个大的数值
7. 如果再malloc一个chunk，得到的chunk指针就是目标位置

因为实际上从fastchunk变成tcachechunk其实就只是改了chunk的fd，所以这个过程应该是：从fastbin取出chunk->改fd->放入tcachebin这样循环，如果期间遇到fd是0的chunk，改完就结束，如果fd非法就会崩溃，但是这个循环只会循环7次（tcachebin就7个位置）

## House of botcake
申请任意地址

准备：

先malloc好7个tcachechunk，用来准备后面填满tcachebin

malloc三个chunk：两个tcachechunk（第一个叫prev，第二个是victim），加上一个gap防止和topchunk合并

1. free掉前七个chunk填满tcachebin
2. free掉victim，进入unsortedbin
3. 再free掉prev让它和victim合并
4. 这时候再malloc一个tcachechunk让tcache空一个位置
5. 再free一遍victim，这时候victim就进到tcachebin里面了
6. 然后接着malloc一块大小够从prev覆盖到victim的chunk，把victim的fd覆盖成目标位置
7. 再把victim给malloc出来，这时候tcachebin指针就指向目标地址了
8. 再malloc一次就可以申请到目标地址了

## House of einherjar
前提：有 heap 地址，有 off by null（one）

申请对齐 0x10 的地址

1. malloc 一个 chunk a （后面用来在里面伪造 fakechunk）
2. 再 malloc 两个 chunk b、c
3. 用 off by null 通过 b 修改到 c 的 size 最低位，让 prev_inuse 位为 0

> 要注意：off by null 需要 b 申请的时候不对齐 0x10，这时候就会复用到 c 的 prev_size 域，从而让输入的数据紧贴 c 的 size 域
>

4. 再在 c 的 prev_size 域伪造一个 fake_prev_size，大小是从 c 的 prev_size 到在 a 伪造的 fakechunk 的 prev_size 域
5. 在 a 里面伪造一个 fakechunk（设置 prev_size=0、size=fake_prev_size，fd 和 bk 都指向 fakechunk 自己的 prev_size 域）
6. 然后填满 tcachebin，让之后 free 的 chunk 进入 fastbin
7. free 掉 c，这时候就让 c 把 b 和 a 里面伪造的 chunk 全部都合并了，放进了 unsortedbin 当中
8. malloc 对应大小获得合并后的 chunk 
9. 然后 malloc 再 free 一个 chunk 当作 padding（高版本的 tcachebin 里面可能是有检查，只有一个 chunk 的时候改 fd 会被检查到）
10. 接着 free 掉 b（b 被包含在合并的 chunk 当中）
11. 通过申请出来的合并的 chunk 来修改 b 的 fd（记得加密`(addr >> 12) ^ ptr`）
12. 然后再 malloc 出来的就是目标地址的 chunk

总结：其实就是利用两个堆块夹一个堆块，改最后一个堆块的 prev_size 和在第一个堆块里面伪造堆块，实现对中间 victim 堆块在 free 后的控制，从而修改 fd 实现任意地址申请

## House of lore
前提：有 heap 地址，可以修改 chunk 的 bk，有内存可以自由修改伪造 chunk

实现任意地址申请

1.  首先 malloc 出 victimchunk （大于 fastchunk）
2. 然后 malloc 7 个 tcachechunk （dummies）准备填充 tcachebin
3. 伪造一个 bin 的 chunk，一共 7 个，只需要伪造 bk 连一块，尾部的 chunk 的 bk 置零
4. 再在栈或者其他地方伪造两个 bufferchunk1、2：buffer1 的 prev_size 和 size 置空，fd 指向 victim，bk 指向 buffer2；buffer2 的 fd 指向 buffer1，bk 指向上面伪造的 bin（freelist）的开头
5. 先申请一个 largechunk 防止 free 时和 topchunk 合并
6. 把 7 个 dummies free 掉，填满 tcachebin
7. 然后再 free 掉 victim 进入 unsortedbin
8. 再 malloc 一个 unsortedbin 和 smallbin 都取不出来的大 chunk 让 victim 进入 smallbin
9. 利用 UAF 或者堆溢出修改 victim 的 bk 指针，指向 buffer1
10. 重新将 tcachebin 当中的所有 chunk malloc 出来
11. 然后再 malloc 就可以把 victimchunk 申请出来
12. 接着 malloc 就可以把伪造堆块的内存申请出来了

## House of spirit
实现伪造对齐 0x10 地址的 chunk 进入 fastbin，进而实现申请任意地址

1. malloc 7 个 tcachechunk 然后全free 填满 tcachebin
2. 伪造一个完整的 chunk （用来申请）和 nextchunk 的 size（size 大小范围 0x10-0x20000）
3. 然后直接 free 就可以进 fastbin 了
4. 要申请出来的话就 calloc 绕过 tcachebin 申请或者先 malloc 7 次把 tcachebin 清空之后再 malloc 出来

> 别忘了 size 是包括 size 域大小的，chunk 指针是指向用户内存的（坦克里是没有后视镜的（？）
>

## Largebin attack
没想到这个版本的 largebin attack 这么简洁

1. 先 malloc 一个 largechunk p1
2. malloc 一个 gap 防止合并
3. 再 malloc 一个比 p1 要小的 largechunk p2
4. 再 malloc 一个 gap 防止合并
5. free 掉 p1
6. 再 malloc 一个大于 p1 的 chunk，让 p1 进入 largebin
7. free 掉 p2，进入 unsortedbin
8. 修改 p1 的 bk_nextsize 为目标地址 target
9. 最后 malloc 一个大于 p1 的 chunk，让 p2 也进入 largebin
10. 这时候就把 target 指向的值写成了 p2 的地址

