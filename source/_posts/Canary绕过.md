---
title: Canary绕过
date: '2025-05-11 22:37:40'
updated: '2025-05-12 13:25:23'
---
canary 是存放在函数栈 rbp-0x8 或 ebp-0x4 位置的数据，在函数开始时压入栈，

在函数返回前会检查它的值是否正确，如果被篡改就会调用__stack_chk_fail 函数

Canary 设计为以字节 \x00 结尾，本意是为了保证 Canary 可以截断字符串

## 字符串输出带出
覆盖 canary 的最低字节，在打印栈上字符串时可以连着 canary 一同带出

但是要注意，换行符可能覆盖到了 canary 的最低字节，记得减去 0xa（换行符 ascii 码）

## 格式化字符串泄露
利用格式化字符串的偏移读取栈上的 canary

## 利用子进程爆破
程序 folk 出来的子进程的 canary 完全一致，

可以利用子进程爆破 canary，一个字节一个字节覆盖，报错则下一个

## 劫持__stack_chk_fail 函数
已知 Canary 失败的处理逻辑会进入到 __stack_chk_failed 函数，__stack_chk_failed 函数是一个普通的延迟绑定函数，可以通过修改 GOT 表劫持这个函数

## 覆盖 TLS 中储存的 Canary 值
已知 Canary 储存在 TLS 中，在函数返回前会使用这个值进行对比。当溢出尺寸较大时，可以同时覆盖栈上储存的 Canary 和 TLS 储存的 Canary 实现绕过。

