---
title: Gadget获取
date: '2025-01-12 20:02:12'
updated: '2025-05-09 17:22:10'
---
## one_gadget
```python
one_gadget libc文件路径
```

![](/images/5ce2dac791e6fe0ca72b39e541251aa7.png)

注意：要满足`execve("/bin/sh",0,0)`才能 getshell

## ROPgadget
### 找 gadget
```shell
ROPgadget --binary 文件名 --only "pop|ret" | grep rdi
```

### 找 strings
```shell
ROPgadget --binary 文件名 --string '/bin/sh'
```

### 静态编译一把梭
```python
ROPgadget --binary 文件名 --ropchain
```

