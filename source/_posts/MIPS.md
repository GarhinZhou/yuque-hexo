---
title: MIPS
date: '2025-11-11 10:45:01'
updated: '2025-12-29 12:45:28'
---
## <font style="background-color:rgba(255, 255, 255, 0);">寄存器</font>
<font style="background-color:rgba(255, 255, 255, 0);">MIPS有32个32位通用寄存器（在MIPS32中；MIPS64为64位），部分寄存器有约定用途：</font>

| <font style="background-color:rgba(255, 255, 255, 0);">寄存器编号</font> | <font style="background-color:rgba(255, 255, 255, 0);">汇编名</font> | <font style="background-color:rgba(255, 255, 255, 0);">用途说明</font> |
| --- | --- | --- |
| `<font style="background-color:rgba(255, 255, 255, 0);">$0</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$zero</font>` | **<font style="background-color:rgba(255, 255, 255, 0);">恒为0</font>**<font style="background-color:rgba(255, 255, 255, 0);">，写入被忽略</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$1</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$at</font>` | **<font style="background-color:rgba(255, 255, 255, 0);">汇编器临时寄存器</font>**<font style="background-color:rgba(255, 255, 255, 0);">（assembler temporary），用户不应直接使用</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$2–$3</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$v0–$v1</font>` | <font style="background-color:rgba(255, 255, 255, 0);">函数返回值（整数）</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$4–$7</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$a0–$a3</font>` | <font style="background-color:rgba(255, 255, 255, 0);">函数参数（前4个）</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$8–$15</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$t0–$t7</font>` | **<font style="background-color:rgba(255, 255, 255, 0);">临时寄存器</font>**<font style="background-color:rgba(255, 255, 255, 0);">，调用者保存（caller-saved），64 位当中前 3 个被用于参数传递的后 3</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$16–$23</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$s0–$s7</font>` | **<font style="background-color:rgba(255, 255, 255, 0);">保存寄存器</font>**<font style="background-color:rgba(255, 255, 255, 0);">，被调用者保存（callee-saved）</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$24–$25</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$t8–$t9</font>` | <font style="background-color:rgba(255, 255, 255, 0);">额外临时寄存器</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$26–$27</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$k0–$k1</font>` | **<font style="background-color:rgba(255, 255, 255, 0);">操作系统保留</font>**<font style="background-color:rgba(255, 255, 255, 0);">，用于中断/异常处理</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$28</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$gp</font>` | <font style="background-color:rgba(255, 255, 255, 0);">全局指针（Global Pointer）存放基地址，通过偏移访问数据</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$29</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$sp</font>` | <font style="background-color:rgba(255, 255, 255, 0);">堆栈指针（Stack Pointer）</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$30</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$fp</font>` | <font style="background-color:rgba(255, 255, 255, 0);">帧指针（Frame Pointer）或需要保存的寄存器（s8）</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">$31</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">$ra</font>` | **<font style="background-color:rgba(255, 255, 255, 0);">返回地址</font>**<font style="background-color:rgba(255, 255, 255, 0);">（Return Address），由</font>`<font style="background-color:rgba(255, 255, 255, 0);">jal</font>`<font style="background-color:rgba(255, 255, 255, 0);">等指令自动写入</font> |


MIPS 有些函数比较简单的就不会用到 fp

## <font style="background-color:rgba(255, 255, 255, 0);">指令</font>
[<font style="background-color:rgba(255, 255, 255, 0);">MIPS 指令集</font>](https://zhida.zhihu.com/search?content_id=215378155&content_type=Article&match_order=1&q=MIPS+%E6%8C%87%E4%BB%A4%E9%9B%86&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NjMwMDE2NzYsInEiOiJNSVBTIOaMh-S7pOmbhiIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjIxNTM3ODE1NSwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.1YEJe6Sjxxed0bHqn8NXCRmkg5K9wIhh8OTk_6GvGgU&zhida_source=entity)<font style="background-color:rgba(255, 255, 255, 0);">总共包含约 111 条指令，每条指令以 32 位表示。MIPS指令的示例如下：</font>

```plain
add $r12, $r7, $r8
```

![](/images/c02597be8931755ad646acdb21826819.png)

<font style="background-color:rgba(255, 255, 255, 0);">上面是 MIPS 加法指令的程序集（上）和二进制（下）表示形式。该指令告诉处理器计算寄存器 7 和 8 中值的总和，并将结果存储在寄存器 12 中。美元符号用于指示寄存器上的操作</font>

<font style="background-color:rgba(255, 255, 255, 0);">彩色二进制表示说明了 MIPS 指令的 6 个字段</font>

<font style="background-color:rgba(255, 255, 255, 0);">处理器通过第一个（粉）和最后一个字段（绿）中的二进制数字来识别指令类型。在这种情况下，处理器会识别出此指令是从其第一个字段中的零和最后一个字段中的 20 开始的加法</font>

<font style="background-color:rgba(255, 255, 255, 0);">操作数以蓝色和黄色字段表示，所需的结果位置显示在第四个（灰色）字段中。橙色字段表示移位量，这是加法运算中不使用的</font>

![](/images/66372e3b8d33d0cb0f673150ed52965b.jpeg)![](/images/c66e56883f6014f61937b85647c74199.jpeg)

`move $sp, $fp`：将右寄存器的值赋给左寄存器

### 栈相关
因为 MIPS 架构没有 POP 和 PUSH 指令，只是通过 SP 对栈进行操作，而且具体入栈的操作也跟 ARM 很像（也有可能是因为 ARM 和 MIPS 都是遵循 RISC 规则）

PUSH：

```cpp
addi $sp, $sp, -12   # 先为 3 个寄存器分配空间（12 字节）
sw   $ra, 8($sp)     # 保存返回地址（高地址）
sw   $s0, 4($sp)     # 保存 $s0
sw   $s1, 0($sp)     # 保存 $s1（栈顶）
```

POP：

```cpp
lw   $s1, 0($sp)     # 恢复 $s1
lw   $s0, 4($sp)     # 恢复 $s0
lw   $ra, 8($sp)     # 恢复 $ra
addi $sp, $sp, 12    # 一次性释放整个栈帧
```

这个栈指针 fp 很有意思啊，实际的作用其实跟 rbp 一模一样，用来链接函数栈帧方便回溯

但是具体的行为又不太一样：

x86 的 rbp 是一直指向保存上一个函数 rbp 的地方的，函数变量统一通过 rbp 的偏移获取

而 mips 这里在函数开始的时候会把 sp 赋给 fp，然后其中有些用到栈上变量的时候会通过 fp 的偏移来获取（有些又是通过 sp 的偏移获取的），直到函数结束之后再把 fp 赋给 sp，再把存在栈上面的上个函数的 fp 赋给 fp

