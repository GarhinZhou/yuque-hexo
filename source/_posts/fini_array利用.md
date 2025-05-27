---
title: fini_array利用
date: '2025-05-11 18:23:11'
updated: '2025-05-12 16:21:08'
---
## _fini_array
_fini_array（或 __fini_array）是一个特殊的ELF符号数组，用于存储在程序或共享对象的终止（清理）阶段将要执行的终止函数的地址

在ELF二进制文件中，除了存储初始化函数数组（.init_array）之外，还可以包含一个终止函数数组（.fini_array）。这些函数会在程序或共享对象退出或终止时以相反的顺序执行，用于进行资源清理、关闭文件描述符、释放内存等操作

程序运行过程：

![](/images/bf632f1b8d9ddebaea8d7ad6668287e4.png)

__libc_csu_fini 函数是 main 函数退出返回到 __libc_start_main 后，通过 __libc_start_main 调用的。具体看看函数：

```c
.text:0000000000402960 __libc_csu_fini proc near               ; DATA XREF: start+F↑o
.text:0000000000402960 ; __unwind {
.text:0000000000402960                 push    rbp
.text:0000000000402961                 lea     rax, unk_4B4100
.text:0000000000402968                 lea     rbp, _fini_array_0
.text:000000000040296F                 push    rbx
.text:0000000000402970                 sub     rax, rbp
.text:0000000000402973                 sub     rsp, 8
.text:0000000000402977                 sar     rax, 3
.text:000000000040297B                 jz      short loc_402996
.text:000000000040297D                 lea     rbx, [rax-1]
.text:0000000000402981                 nop     dword ptr [rax+00000000h]
.text:0000000000402988
.text:0000000000402988 loc_402988:                             ; CODE XREF: __libc_csu_fini+34↓j
.text:0000000000402988                 call    qword ptr [rbp+rbx*8+0]
.text:000000000040298C                 sub     rbx, 1
.text:0000000000402990                 cmp     rbx, 0FFFFFFFFFFFFFFFFh
.text:0000000000402994                 jnz     short loc_402988
.text:0000000000402996
.text:0000000000402996 loc_402996:                             ; CODE XREF: __libc_csu_fini+1B↑j
.text:0000000000402996                 add     rsp, 8
.text:000000000040299A                 pop     rbx
.text:000000000040299B                 pop     rbp
.text:000000000040299C                 jmp     sub_48E32C
.text:000000000040299C ; } // starts at 402960
.text:000000000040299C __libc_csu_fini endp
```

这三行源码，是劫持 fini_array 实现无限写进行 ROP 的关键：

```c
//将 fini_array[0] 的值加载到 rbp
.text:0000000000402968                 lea     rbp, _fini_array_0
//经过一系列运算后，这里会 call fini_array[1] ，也就是调用存储在 fini_array[1] 的指针
.text:0000000000402988                 call    qword ptr [rbp+rbx*8+0]
//调用完 fini_array[1] 之后再次进过一系列运算，这里会 call fini_array[0]
.text:0000000000402988                 call    qword ptr [rbp+rbx*8+0]
```

看一下 fini_array 的代码：

```c
.fini_array:00000000004B40F0 _fini_array     segment para public 'DATA' use64
.fini_array:00000000004B40F0                 assume cs:_fini_array
.fini_array:00000000004B40F0                 ;org 4B40F0h
.fini_array:00000000004B40F0 _fini_array_0   dq offset sub_401B00    ; DATA XREF: .text:000000000040291C↑o
.fini_array:00000000004B40F0                                         ; __libc_csu_fini+8↑o
.fini_array:00000000004B40F8                 dq offset sub_401580
.fini_array:00000000004B40F8 _fini_array     ends
```

fini_array 里面存储了两个指针，调用顺序为：先 fini_array[1] ，再 fini_array[0] 。那么如果把 fini_array[1] 覆盖为函数 A 的地址，fini_array[0] 覆盖为 __libc_csu_fini 的地址，当退出 main 后，程序会一直循环持续到 fini_array[0] 被覆盖为其他值

将 rbp 的值修改为 fini_array[0] 所在的地址，那么配合 `leave;ret` 就能将栈迁移到 fini_array + 0x10 的地址



在 pwndbg 当中可以通过指令：`elf`来查看整个程序 elf 段的分布

![](/images/731e0d2341aae8a85f08b6a092187b23.png)

如果是非栈上格式化字符串，暂且第一次只能够修改_fini_array[0]为函数A，此时第一次main ret后会回到A，但是此时栈帧并不是第一次main的栈帧，而是在上面（较低地址处），而且ret会返回到一个libc地址（之后补充图片）。我们可以部分写这个libc为one_gadget来getshell

