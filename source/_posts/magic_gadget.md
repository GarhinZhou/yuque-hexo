---
title: magic_gadget
date: '2025-05-12 16:38:25'
updated: '2025-05-12 16:48:16'
---
magic_gadget 实际上是pwn攻击中一些具有奇妙功效的gadget，它们功能各异，但是应用在pwn的payload构造中往往是一个与众不同但有方便快捷的构造方式

## 0x01 add dword ptr [rbp - 0x3d], ebx ; nop ; ret ;
```c
add dword ptr [rbp - 0x3d], ebx ; nop ; ret ;
```

这个gadget的功能不难理解，就是将ebx寄存器中的值加到rbp-0x3d的位置上，乍一看似乎很难用，需要控制rbp和ebx才能实现半个任意地址写的功能，但是在ROP中有一个很高效使用的方法控制这两个寄存器：ret2csu

```c
.text:000000000040059A                 pop     rbx
.text:000000000040059B                 pop     rbp
.text:000000000040059C                 pop     r12
.text:000000000040059E                 pop     r13
.text:00000000004005A0                 pop     r14
.text:00000000004005A2                 pop     r15
.text:00000000004005A4                 retn
```

