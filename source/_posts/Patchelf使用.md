---
title: Patchelf使用
date: '2024-12-14 17:56:21'
updated: '2025-05-09 17:22:16'
---
`patchelf`是一个用来修改 ELF（Executable and Linkable Format，执行和可链接格式）文件的工具。ELF 是 Linux 和类 Unix 系统中常见的可执行文件格式，`patchelf` 允许你对 ELF 文件进行一些常见的修改操作，而不需要重新编译源代码。、

先补充一点刚学的知识：

动态链接库（共享库）是指libc库libc.so.6文件，动态链接器是指ld（loader）开头的so.2文件，要保证动调时程序能运行，要同时配置好这两个

## 安装步骤
[来自CSDN](https://blog.csdn.net/gitblog_07485/article/details/142221877)

### 0x01 克隆项目源码
首先，使用 Git 从 GitHub 克隆 patchelf 项目源码：

`git clone [https://github.com/NixOS/patchelf.git](https://github.com/NixOS/patchelf.git)`

`cd patchelf`

### 0x02 生成构建文件
运行 bootstrap.sh 脚本以生成构建所需的文件：

`./bootstrap.sh`

在这一步遇到了一个报错：

`./bootstrap.sh: 2: autoreconf: not found`

查了一下发现是Linux系统缺少autoreconf工具：

安装一下：

`sudo apt install -y autoconf automake libtool`

安装完之后再运行bootstrap.sh脚本就成功了

### 0x03 配置构建环境
运行 configure 脚本来配置构建环境：

`./configure`

### 0x04 编译项目
使用 make 命令编译项目：

`make`

### 0x05 运行测试
编译完成后，运行测试以确保 patchelf 正常工作：

`make check`

### 0x06 安装 patchelf
如果测试通过，用以下命令将 patchelf 安装到系统中：

`sudo make install`

## 使用方法
### 0x01 修改ELF文件的动态链接器(ld 文件)
通过 patchelf，你可以修改 ELF 文件的动态链接器（也称为解释器），通常用于修改程序的加载行为。例如，将程序的加载器改为不同版本的 libc。

```python
patchelf --set-interpreter /path/to/interpreter my_program
```

<font style="background-color:#FBDE28;">注意：ld 文件名类似</font>`<font style="background-color:#FBDE28;">ld-2.27.so</font>`

### 0x02 修改 ELF 文件的 RPATH 或 RUNPATH(libc 路径)
RPATH 和 RUNPATH 是 ELF 文件中的动态链接库搜索路径。你可以使用 patchelf 来修改这些路径，从而控制程序在运行时如何找到共享库。

`patchelf --set-rpath /new/library/path my_program`

注意：

RPATH 和 RUNPATH 的区别在于，RPATH 会在 LD_LIBRARY_PATH 中的路径之前被搜索，而 RUNPATH 会在之后。

<font style="background-color:#FBDE28;">注意是路径，一系列 libc 文件存在的路径</font>

### 0x03 修改 ELF 文件的 SONAME（一般为空）
SONAME 是共享库的名称(例如`libc.so.6`)，通常用于链接器在运行时查找和加载共享库。你可以用 patchelf 修改 ELF 文件的 SONAME，这种修改通常在共享库的版本控制中非常有用。

`patchelf --set-soname new_soname.so my_program`

### 0x04 改变文件的执行权限
使用 patchelf 可以将 ELF 文件的运行权限修改为可执行文件，或者移除其执行权限。

`patchelf --add-needed my_library.so my_program`

### 0x05 查看 ELF 文件的信息
patchelf 也提供了查看 ELF 文件的基本信息功能，比如查看文件的解释器、RPATH、SONAME 等。

`patchelf --print-interpreter my_program`

`patchelf --print-rpath my_program`

`patchelf --print-soname my_program`

### 0x06 修改 ELF 文件加载的 libc.so 文件
使用 --replace-needed这个选项将旧的 libc.so 替换成要加载的 libc.so

```python
patchelf --replace-needed libc.so.6 libc-版本.so文件路径 ./elf
```

在使用 --replace-needed 时，第 2 个参数是程序原本的动态库的路径，可以由 ldd $目标文件 得到，第 3 个参数是新的动态库的路径，第 4 个参数为要修改文件的路径。

根据上面 ldd 的结果，可以知道 ELF 中的 libc 的路径为 "libc.so.6"，所以替换 libc 时所使用的第 2 个参数为 "libc.so.6"

## 注意的点
[来自c_lby师傅的博客](https://c-lby.top/2024/2023newstar-week4-wp/)

```protobuf
$ patchelf --set-interpreter ~/glibc-all-in-one/libs/2.23-0ubuntu3_amd64/libc.so.6 --set-rpath ~/glibc-all-in-one/libs/2.23-0ubuntu3_amd64/ Double
```

假如想用以上命令patch Double这个程序，运行这个程序的时候就会变成这样：

![](/images/bfd89f5e39f4bac8727de6f9695ee2d7.png)

只要将两个参数分开执行就行，也就是：

```protobuf
$ patchelf --set-interpreter ~/glibc-all-in-one/libs/2.23-0ubuntu3_amd64/ld-linux-x86-64.so.2 Double
$ patchelf --set-rpath ~/glibc-all-in-one/libs/2.23-0ubuntu3_amd64 Double
```

