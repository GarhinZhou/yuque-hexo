---
title: HDC使用
date: '2025-07-15 18:24:10'
updated: '2025-07-15 21:22:24'
---
HDC 全称是 OpenHarmony device connector，类似 ADB？

## 常用指令
看可连接目标：

```shell
./hdc list targets [-v] #详细信息
```

可以查看到连接上的目标的连接密钥

执行指令：

```shell
./hdc -t connect-key command
```

```shell
./hdc shell [-b bundlename] [command]
```

安装 APP 包：

```shell
./hdc shell [-b bundlename] [command]
```

| <font style="color:rgb(36, 39, 40);">src</font> | <font style="color:rgb(36, 39, 40);">应用安装包的文件名</font> |
| :--- | :--- |
| <font style="color:rgb(36, 39, 40);">-r</font> | <font style="color:rgb(36, 39, 40);">替换已存在应用（.hap）</font> |
| <font style="color:rgb(36, 39, 40);">-s</font> | <font style="color:rgb(36, 39, 40);">安装一个共享包（.hsp）</font> |


卸载 APP 包：

```shell
./hdc uninstall [-k|-s] packageName
```

| **<font style="color:rgb(36, 39, 40);">参数名</font>** | **<font style="color:rgb(36, 39, 40);">说明</font>** |
| :--- | :--- |
| <font style="color:rgb(36, 39, 40);">packageName</font> | <font style="color:rgb(36, 39, 40);">应用安装包。</font> |
| <font style="color:rgb(36, 39, 40);">-k</font> | <font style="color:rgb(36, 39, 40);">保留/data和/cache目录。</font> |
| <font style="color:rgb(36, 39, 40);">-s</font> | <font style="color:rgb(36, 39, 40);">卸载共享包。</font> |


[HDC 常用指令](https://developer.huawei.com/consumer/cn/training/course/slightMooc/C101717494752698457)

