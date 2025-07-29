---
title: Seccomp
date: '2025-01-24 19:03:24'
updated: '2025-07-29 15:09:57'
---
限制进程进行系统调用用的函数`seccomp()`，会限制程序自身进程和子进程

## `<font style="color:rgb(36, 41, 47);">seccomp()</font>`
+ 函数名解释：
    - "sec": 代表 "security"（安全）的缩写，
    - "comp": 代表"computation"（计算）或 "compatibility"（兼容性）的缩写
+ 功能：
    - `seccomp` 是一个系统调用，用来设置 seccomp 模式，即安全计算模式。它通过过滤进程的系统调用来实现安全性控制，防止进程访问不必要或危险的系统调用，从而减少潜在的攻击面
    - 在`seccomp` 模式下，进程只能执行一个受限的系统调用集合，其余的系统调用会被拒绝

```c
/*
 * seccomp actions
 */

/**
 * Kill the process
 */
#define SCMP_ACT_KILL		0x00000000U
/**
 * Throw a SIGSYS signal
 */
#define SCMP_ACT_TRAP		0x00030000U
/**
 * Return the specified error code
 */
#define SCMP_ACT_ERRNO(x)	(0x00050000U | ((x) & 0x0000ffffU))
/**
 * Notify a tracing process with the specified value
 */
#define SCMP_ACT_TRACE(x)	(0x7ff00000U | ((x) & 0x0000ffffU))
/**
 * Allow the syscall to be executed after the action has been logged
 */
#define SCMP_ACT_LOG		0x7ffc0000U
/**
 * Allow the syscall to be executed
 */
#define SCMP_ACT_ALLOW		0x7fff0000U
```

## `<font style="color:rgb(36, 41, 47);">prctl()</font>`
`prctl`主要实现了两类命令，`SET` 和 `GET` ， 即操作进程运行时和获取进程信息

+ 函数名解释：
    - "pr": 代表 "process"（进程）的缩写，
    - "ctl": 代表 "control"（控制）的缩写，意味着该函数可以用来控制或调整进程的行为、状态或属性
+ 功能：
    - `prctl`是一个非常灵活的系统调用，允许进程设置、获取或修改自己的某些属性。它可以用于控制许多进程特性，例如：
        * 设置进程的资源限制（如内存、CPU 时间等）
        * 控制进程的行为（如进程名、调试选项等）
        * 配置与 `seccomp` 相关的安全模式，设置进程的安全计算模式（例如，通过`PR_SET_SECCOMP`设置 seccomp 策略）

`<font style="color:rgb(36, 41, 46);">seccomp</font>`<font style="color:rgb(36, 41, 46);">就是基于</font>`<font style="color:rgb(36, 41, 46);">prctl</font>`<font style="color:rgb(36, 41, 46);">实现的</font>

```c
case PR_SET_SECCOMP:
error = prctl_set_seccomp(arg2, (char __user *)arg3);
```

这里涉及到这样一条调用链：

```c
-->prctl 
	-->prctl_set_seccomp
		-->do_seccomp
			-->seccomp_set_mode_filter
				--> seccomp_attach_filter
```

seccomp_attach_filter 核心代码如下：

```c
filter->prev = current->seccomp.filter;
seccomp_cache_prepare(filter);
current->seccomp.filter = filter;
atomic_inc(&current->seccomp.filter_count);
```

current是一个全局的指针，指向当前进程的task结构体，主要保存了当前进程的一些信息

所以，注册seccomp，实际上就是设置了当前进程的filter规则

## seccomp-tools 使用
`seccomp()`会产生 BPF 文件，

```bash
seccomp-tools <指令> [文件路径/PID]
```

+ asm  将存有 Seccomp 规则的 txt 文件汇编成 BDF 文件
+ disasm  将 BDF 文件反汇编成 txt 文件
+ dump  自动从可执行文件当中获取 Seccomp BDF 文件
+ emu   模拟 Seccomp 规则

### 查看可执行文件 seccomp 规则
可以查看的前提是文件会被执行一遍，并且 seccomp 规则在执行的时候被制定了

```c
seccomp-tools dump ./binary
```

结果例子：

```c
pwner@GarhinCTFer:~/FocusingCTF$ seccomp-tools dump ./shellcode
Welcome LitCTF 2025
Please input your shellcode:
 line  CODE  JT   JF      K
=================================
 0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x00 0x06 0xc000003e  if (A != ARCH_X86_64) goto 0008
 0002: 0x20 0x00 0x00 0x00000000  A = sys_number
 0003: 0x35 0x00 0x01 0x40000000  if (A < 0x40000000) goto 0005
 0004: 0x15 0x00 0x03 0xffffffff  if (A != 0xffffffff) goto 0008
 0005: 0x15 0x01 0x00 0x00000000  if (A == read) goto 0007
 0006: 0x15 0x00 0x01 0x00000002  if (A != open) goto 0008
 0007: 0x06 0x00 0x00 0x7fff0000  return ALLOW
 0008: 0x06 0x00 0x00 0x00000000  return KILL
```

