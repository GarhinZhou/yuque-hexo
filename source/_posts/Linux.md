---
title: Linux
date: '2024-12-17 11:01:27'
updated: '2025-12-14 20:28:07'
---
## 换源
## 特殊文件、路径
`proc/self`当中的`environ`是环境变量，动态 flag 经常出现在这里

## 快捷键和常用指令
`bash`当中可以使用快捷键`Tab`补全文件名或者命令名

`sudo su` 切换到 root 用户

`ctrl+d` 退出 root 用户；也可以表示eof，不需要回车

`ctrl+h`在文件管理器可以通过这个快捷键查看以`.`开头的隐藏文件

## 流
&号后面接文件描述符是指流向文件描述符

```python
cat flag >> &1
```

# ubuntu24 调试抽风处理
清空对应系统日志：

```cpp
sudo truncate -s 0 /var/log/syslog
```

查找对应进程：

```c
ps aux | grep [进程名]
```

杀死对应进程：

```c
kill -9 [pid]
```

# TTY
TTY 是 Teletype（电传打字机）的缩写，但是逐渐演变成终端的含义

在现代 Linux 系统中，即便没有外接终端，系统也会模拟出多个 TTY

可以通过快捷键 Ctrl + Alt + F1 到 F6 在不同的 虚拟控制台 之间切换

> 但是有用户界面 GUI 就没必要了...试了试 F1 我屏幕直接不显示了）
>

每个控制台都是一个独立的 TTY（例如 /dev/tty1）

