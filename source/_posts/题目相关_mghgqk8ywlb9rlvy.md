---
title: 题目相关
date: '2025-10-11 23:00:13'
updated: '2025-12-28 18:11:03'
---
内核没开 kaslr 的情况下会从`0xffffffff81000000`开始加载，就相当于没开 PIE 的情况下程序基址在 0x400000 开始

有些时候题目给的启动脚本启动不了可能是因为设置的虚拟机大小太小，可以适当地调大一点

查看系统内核版本：

```bash
uname -r
```

![](/images/8b9a64f9bb33187c5b159d86037bcfb9.png)

# 常用文件
`/proc/kallsyms`是内核符号表

`/proc/sys/kernel/kptr_restrict`和`/proc/sys/kernel/dmesg_restrict`这类都是内核配置

其中`/proc/sys/kernel/kptr_restrict`：设为 1 就不能通过 `/proc/kallsyms` 查看函数地址

![](/images/a35bd029bd43afb1b0618e5f0c51d4e5.png)

`/proc/sys/kernel/dmesg_restrict`：设为 1 就不能通过 `dmesg` 查看 kernel 的信息



# 上传脚本
因为 kernel pwn 大部分时候都是给了 shell，需要提权到 root 权限读 root 路径下的 flag

所以需要通过脚本传 elf 文件过去

通过把 elf base64 编码后传输：

```c
from pwn import *
import base64
#context.log_level = "debug"

with open("./exp", "rb") as f:
    exp = base64.b64encode(f.read())

p = remote("127.0.0.1", 11451)
#p = process('./run.sh')
p.sendline()
p.recvuntil("/ $")

count = 0
for i in range(0, len(exp), 0x200):
    p.sendline("echo -n \"" + exp[i:i + 0x200].decode() + "\" >> /tmp/b64_exp")
    count += 1
    log.info("count: " + str(count))

for i in range(count):
    p.recvuntil("/ $")

p.sendline("cat /tmp/b64_exp | base64 -d > /tmp/exploit")
p.sendline("chmod +x /tmp/exploit")
p.sendline("/tmp/exploit ")

p.interactive()
```

# 编译器 musl-gcc
因为做内核题都是要用脚本上传一个 C 语言 exp，所以这个 exp 的大小要尽可能的小，这样上传的时间会大大缩短，这个 musl-gcc 库真的很恐怖，编译出来的 elf 能够比 glibc 小好多倍

安装：

```python
sudo apt install musl-tools
```

用法其实跟 gcc 很像

