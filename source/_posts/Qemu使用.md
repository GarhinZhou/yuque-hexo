---
title: Qemu使用
date: '2025-02-09 23:26:57'
updated: '2025-12-17 15:24:11'
---
## 安装 qemu
```bash
sudo apt install qemu-user
sudo apt install qemu-user-static
sudo apt install qemu-system
```

## 启用多架构
Debian/Ubuntu 支持 Multiarch，可以直接安装其他架构的官方库包

```bash
# 查看当前支持的架构
dpkg --print-foreign-architectures

# 添加目标架构（例如 aarch64）
sudo dpkg --add-architecture arm64
# 删除目标结构
sudo dpkg --remove-architecture arm64

# 更新软件源
sudo apt update
sudo apt upgrade
```

其中不同架构的名称：

+ `<font style="background-color:rgba(175, 184, 193, 0.2);">arm64</font>` → AArch64 (ARMv8)
+ `<font style="background-color:rgba(175, 184, 193, 0.2);">armhf</font>` → ARM hard-float (ARMv7)
+ `<font style="background-color:rgba(175, 184, 193, 0.2);">mipsel</font>` / `<font style="background-color:rgba(175, 184, 193, 0.2);">mips64el</font>` → 小端 MIPS
+ `<font style="background-color:rgba(175, 184, 193, 0.2);">ppc64el</font>` → PowerPC 64-bit little-endian

## 安装不同架构的基础库
```bash
apt search "libc6-" | grep "arm"
sudo apt install libc6-arm64-cross
sudo apt install libc6-dbg-arm64-cross
```

然后就可以通过下面的指令来运行不同架构的程序了：

```bash
qemu-aarch64 -L /usr/aarch64-linux-gnu/ ./arm64overflow
```

## patch 可执行文件
实际上不同架构的 elf 文件的差别并没有想象中那么大，文件格式实际上是一样的，只是机器码不同

patch 实质上就是对二进制文件里面的文本替换，所以是可以给别的架构的 elf patch 的

比如遇到：

```plain
$ qemu-arm -L /usr/arm-linux-gnueabihf/lib/ ret2win_armv5
qemu-arm: Could not open '/lib/ld-linux.so.3': No such file or directory
```

就可以把 elf patch 到安装好的对应架构的库的路径

## 向 qemu 中写入文件
宿主机在要写入文件的路径下用 python 开一个简易 http 服务：

```bash
python3 -m http.server 8000
```

qemu 虚拟机中使用 wget（busybox 也有）：

```bash
# 10.0.2.2 是 QEMU 默认分配给宿主机的 IP
wget http://10.0.2.2:8000/exp.c
```

## 调试
先装好 gdb-multiarch

```plain
sudo apt install gdb-multiarch
```

在启动 qemu 的时候加上-g 参数和端口，就可以通过`localhost:端口`实现本地调试

```plain
qemu-arm -L /usr/arm-linux-gnueabihf/ -g 6666 ./ret2win_armv5
```

需要两个终端，一个负责通过 qemu 启动程序（可以直接启动，也可以通过 pwntools 脚本启动）

```plain
link=process(['qemu-mips','-L','/usr/mipsel-linux-gnu','-g','6666','./ret2win_mipsel'])
```

另一个通过`gdb-multiarch ./pwn`->`target remote localhost:端口`连接上 qemu 里面的程序

然后就可以开始调试了，不过不知道为什么连接上之后如果没有断点的话没法拦下来程序执行，ctrl+c 会退出连接

