---
title: 获取Libc版本
date: '2024-12-14 20:38:06'
updated: '2025-05-12 16:25:31'
---
## 本机版本
用`ldd --version`

## 靶机版本
比较新的libc库查找网址：[libc_database_search_new](https://libcdb.dariopetrillo.it/)

libcsearcher相同结果的网址：[libc_database_search](https://libc.rip/)

通过泄露函数地址的最后三位进行查询

注意：函数`_libc_start_call_main_`

在查找libc版本时要用`_libc_start_main_ret_`的名称

## 文件版本
`strings libc.so.6 | grep GLIBC`

在`libc.so.6`文件中找到GLIBC的信息即libc版本

## 查看文件对应链接 libc 版本
```bash
ldd <可执行文件或共享库>
```

输出示例：

```bash
linux-vdso.so.1 (0x00007ffd1b7fe000)
libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007fe1d3c70000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fe1d3892000)
/lib64/ld-linux-x86-64.so.2 (0x00007fe1d3e8f000)
```

如果动态链接的可执行文件对应的 libc 库找不到

ldd 的结果会是：

```bash
linux-vdso.so.1 (0x00007ffc9c7fc000)
./libseccomp.so => not found
./libc-2.31.so => not found
```

会报出：No such file or directory 的错误

![](/images/bdb1821a5f5c412ff6fdda7cc0ed2537.jpeg)

## 不同 ubuntu 版本对应 libc 版本
Ubuntu 16.04 <font style="background-color:#FBDE28;">LTS</font> (Xenial Xerus)：<font style="background-color:#FBDE28;">2.23</font>

Ubuntu 16.10 (Yakkety Yak)：2.24

Ubuntu 17.04 (Zesty Zapus)：2.24

Ubuntu 17.10 (Artful Aardvark)：2.26

Ubuntu 18.04 <font style="background-color:#FBDE28;">LTS</font> (Bionic Beaver)：<font style="background-color:#FBDE28;">2.27</font>

Ubuntu 18.10 (Cosmic Cuttlefish)：2.28

Ubuntu 19.04 (Disco Dingo)：2.28

Ubuntu 19.10 (Eoan Ermine)：2.29

Ubuntu 20.04 <font style="background-color:#FBDE28;">LTS</font> (Focal Fossa)：<font style="background-color:#FBDE28;">2.31</font>

Ubuntu 20.10 (Groovy Gorilla)：2.32

Ubuntu 21.04 (Hirsute Hippo)：2.33

Ubuntu 21.10 (Impish Indri)：2.34

Ubuntu 22.04 <font style="background-color:#FBDE28;">LTS</font> (Jammy Jellyfish)：<font style="background-color:#FBDE28;">2.35</font>

Ubuntu 22.10 (Kinetic Kudu)：2.36

Ubuntu 23.04 (Lunar Lobster)：2.37

Ubuntu 23.10 (Mantic Minotaur)：2.38

Ubuntu 24.04 <font style="background-color:#FBDE28;">LTS</font> (Noble Numbat)：<font style="background-color:#FBDE28;">2.39</font>

Ubuntu 25.04 (Plucky Puffin)：2.41（2025.4 预计）

