---
title: Pwndbg使用
date: '2024-12-09 23:32:34'
updated: '2025-07-27 23:33:21'
---
[来自博客](https://www.cnblogs.com/murkuo/p/15965270.html)

## 基本指令
help //帮助

i //info，查看一些信息，只输入info可以看可以接什么参数，下面几个比较常用

i b //常用，info break 查看所有断点信息（编号、断点位置）

i r //常用，info registers 查看各个寄存器当前的值

i f //info function 查看所有函数名，需保留符号

show //和info类似，但是查看调试器的基本信息，如：

show args //查看参数

rdi //常用，+寄存器名代表一个寄存器内的值，用在地址上直接相当与一个十六进制变量

backtrace //查看调用栈

q //quit 退出，常用

vmmap //内存分配情况



## 执行指令
s //单步步入，遇到调用跟进函数中，相当于step into，源码层面的一步

si //常用，同s，汇编层面的一步

n //单步补过，遇到函数不跟进，相当于step over，源码层面的一步

ni //常用，同n，汇编层面的一步

c //continue，常用，继续执行到断点，没断点就一直执行下去

r //run，常用，重新开始执行

start // 类似于run，停在main函数的开始



## 断点指令
### 下普通断点指令b(break)：
b *(0x123456) //常用，给0x123456地址处的指令下断点

<font style="background-color:rgba(255, 255, 255, 0);">b *$rebase(0x123456) //$rebase 在调试开PIE的程序的时候可以直接加上程序的随机地址</font>

b fun_name //常用，给函数fun_name下断点，目标文件要保留符号才行

b file_name:fun_name

b file_name:15 //给file_name的15行下断点，要有源码才行

b 15

b +0x10 //在程序当前停住的位置下0x10的位置下断点，同样可以-0x10，就是前0x10

break fun if $rdi==5 //条件断点，rdi值为5的时候才断

### 删除、禁用断点：
info break(简写: i b) //查看断点编号

delete 5 //常用，删除5号断点，直接delete不接数字删除所有

disable 5 //常用，禁用5号断点

enable 5 //启用5号断点

clear //清除下面的所有断点

### 内存断点指令watch：
watch 0x123456 //0x123456地址的数据改变的时候会断

watch a //变量a改变的时候会断

info watchpoints //查看watch断点信息

### 捕获断点catch：
catch syscall //syscall系统调用的时候断住

tcatch syscall //syscall系统调用的时候断住，只断一次

info break //catch的断点可以通过i b查看

除syscall外还可以使用的有：

1. throw: 抛出异常
2. catch: 捕获异常
3. exec: exec被调用
4. fork: fork被调用
5. vfork: vfork被调用
6. load: 加载动态库
7. load libname: 加载名为libname的动态库
8. unload: 卸载动态库
9. unload libname: 卸载名为libname的动态库
10. syscall [args]: 调用系统调用，args可以指定系统调用号，或者系统名称



## 打印指令
### 查看内存指令x：
x /nuf 0x123456 //常用，x指令的格式是：x空格/nfu，nfu代表三个参数

+ n代表显示几个单元（而不是显示几个字节，后面的u表示一个单元多少个字节），放在/后面
+ u代表一个单元几个字节，b(一个字节)，h(2字节)，w(四字节)，g(八字节)
+ f代表显示数据的格式，f和u的顺序可以互换，也可以只有一个或者不带n，用的时候很灵活
+ x 按十六进制格式显示变量。
+ d 按十进制格式显示变量。
+ u 按十六进制格式显示无符号整型。
+ o 按八进制格式显示变量。
+ t 按二进制格式显示变量。
+ a 按十六进制格式显示变量。
+ c 按字符格式显示变量。
+ f 按浮点数格式显示变量。
+ s 按字符串显示。
+ b 按字符显示。
+ i 显示汇编指令。

x /10gx 0x123456 //常用，从0x123456开始每个单元八个字节，十六进制显示10个单元的数据

x /10xd $rdi //从rdi指向的地址向后打印10个单元，每个单元4字节的十进制数

x /10i 0x123456 //常用，从0x123456处向后显示十条汇编指令

### 打印指令p(print)：
p fun_name //打印fun_name的地址，需要保留符号

p 0x10-0x08 //计算0x10-0x08的结果

p &a //查看变量a的地址

p *(0x123456) //查看0x123456地址的值，注意和x指令的区别，x指令查看地址的值不用星号

p $rdi //显示rdi寄存器的值，注意和x的区别，这只是显示rdi的值，而不是rdi指向的值

p *($rdi) //显示rdi指向的值

### 打印汇编指令disass(disassemble)：
disass 0x123456 //显示0x123456前后的汇编指令

x /10i //我一般喜欢用x显示指令

### 打印源代码指令list：
list //查看当前附近10行代码，要有源码，list指令pwn题中几乎不用，但为了完整性还是简单举几个例子

list 38 //查看38行附近10行代码

list 1,10 //查看1-10行

list main //查看main函数开始10行



## 修改和查找指令
### 修改数据指令set：
set $rdi=0x10 //把rdi寄存器的值变为0x10

set *(0x123456)=0x10 //0x123456地址的值变为0x10，注意带星号

set args "abc" "def" "gh" //给参数123赋值

set args "python -c 'print "1234\x7f\xde"'"' //使用python给参数赋值不可见字符

### 查找数据
search rdi //从当前位置向后查包含rdi的指令，返回若干

search -h //查看search帮助，我也不太长用这个指令

find "hello" //查找hello字符串，pwndbg独有

ropgadget //查找ropgadget，pwndbg独有，没啥用，可以用其他工具

## 堆操作指令（pwndbg插件独有）
arena //显示arena的详细信息

arenas //显示所有arena的基本信息

bins //常用，查看所有种类的堆块的链表情况

fastbins //单独查看fastbins的链表情况

largebins //同上，单独查看largebins的链表情况

smallbins //同上，单独查看smallbins的链表情况

unsortedbin //同上，单独查看unsortedbin链表情况

tcachebins //同上，单独查看tcachebins的链表情况

tcache //查看tcache详细信息

heap //数据结构的形式显示所有堆块，会显示一大堆



下面几个不知道是不是更新掉了，没有定义：

arenainfo //好看的显示所有arena的信息

heapbase //查看堆起始地址

heapinfo、heapinfoall //显示堆得信息，和bins的挺像的，没bins好用

parseheap //显示堆结构，很好用

tracemalloc //好用，会跟提示所有操作堆的地方

### <font style="background-color:#FBDE28;">重点！！！</font>
<font style="background-color:#FBDE28;">遇到在 python 脚本里面调用</font>`<font style="background-color:#FBDE28;">gdb.attach(link)</font>`<font style="background-color:#FBDE28;"> 动调的时候没有符号表，使用 heap、bins 指令时报错：</font>

![](/images/71b1c2d40804471d657e415445357b71.jpeg)

可以手动导入对应的 libc 路径，这样符号就会恢复，可以正常调试了

![](/images/6800b570e57220ce7082e5044d4d2d05.jpeg)

> 弄了我整整两天时间卡在这里啊...（大哭）
>

当然，为了脚本调试起来更方便，肯定不能每次进去都得自己手动输入这条指令嘛

（注意要写绝对地址，不是相对地址）

```shell
gdb.attach(link,"set solib-search-path /rpath") #其实就是patchelf里面用到的rpath
```

## 多线程调试指令
### 查看线程列表
```c
info thread
```

### 切换线程
```c
thread 线程序号
```



## 其他pwndbg插件独有指令
cyclic 50 //生成50个用来溢出的字符，如：aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama

$rebase //开启PIE的情况的地址偏移

b *$rebase(0x123456) //断住PIE状态下的二进制文件中0x123456的地方

codebase //打印PIE偏移，与rebase不同，这是打印，rebase是使用

stack //查看栈

retaddr //打印包含返回地址的栈地址

canary //直接看canary的值

plt //查看plt表

got //查看got表

hexdump //像IDA那样显示数据，带字符串

## 




