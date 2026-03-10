---
title: 绕过KPTI
date: '2025-12-18 14:51:16'
updated: '2025-12-28 10:40:33'
---
# KPTI
内核页表隔离（Kernel page-table isolation），内核空间与用户空间分别使用两组不同的页表集

同时还令内核页表中属于用户地址空间的部分不再拥有执行权限，这使得 `ret2usr` 彻底成为过去式 

开启了 KPTI（内核页表隔离），不能像之前那样直接 swapgs ; iret 返回用户态，而是在返回用户态之前还需要将用户进程的页表给切换回来

> Linux 采用四级页表结构（PGD->PUD->PMD->PTE）
>

```c
页表的层级结构目前发展为如下所示:
+-----+
| PGD |
+-----+
   |
   |   +-----+
   +-->| P4D |
       +-----+
          |
          |   +-----+
          +-->| PUD |
              +-----+
                 |
                 |   +-----+
                 +-->| PMD |
                     +-----+
                        |
                        |   +-----+
                        +-->| PTE |
                            +-----+
      PMD
--> +-----+           PTE
    | ptr |-------> +-----+
    | ptr |-        | ptr |-------> PAGE
    | ptr | \       | ptr |
    | ptr |  \        ...
    | ... |   \
    | ptr |    \         PTE
    +-----+     +----> +-----+
                       | ptr |-------> PAGE
                       | ptr |
                         ...
```

+ pte, pte_t, pteval_t = 页表项
+ pmd, pmd_t, pmdval_t = 页中间目录(Page Middle Directory)
+ pud, pud_t, pudval_t = 页上级目录(Page Upper Directory) 
+ p4d, p4d_t, p4dval_t = 页四级目录(Page Level 4 Directory) 
+ pgd, pgd_t, pgdval_t = 页全局目录(Page Global Directory)

# 页表切换
`CR3` 控制寄存器用以存储当前的 PGD 的地址，因此在开启 KPTI 的情况下用户态与内核态之间的切换便涉及到 `CR3` 的切换，为了提高切换的速度，内核将内核空间的 PGD 与用户空间的 PGD 两张页全局目录表放在一段连续的内存中（两张表，一张一页 4k，总计 8k，内核空间的在低地址，用户空间的在高地址），这样只需要将 `CR3` 的第 13 位取反便能完成页表切换的操作

![](/images/7caccca43460d9d9853c3aef68f37c00.png)

在这两张页表上都有着对用户内存空间的完整映射，但在用户页表中只映射了少量的内核代码（例如系统调用入口点、中断处理等），而只有在内核页表中才有着对内核内存空间的完整映射，但两张页表都有着对用户内存空间的完整映射，如下图所示，左侧是未开启 KPTI 后的页表布局，右侧是开启了 KPTI 后的页表布局

![](/images/bde265b6bac6c0faa8893c98021b9ab3.png)

