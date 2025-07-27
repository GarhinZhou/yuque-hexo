---
title: ptrace及调试
date: '2025-07-26 00:32:04'
updated: '2025-07-28 00:04:02'
---
`ptrace()`系统调用的函数如下：

```c
#include <sys/ptrace.h>       
long ptrace(enum __ptrace_request request, pid_t pid, void *addr, void *data);
```

对函数签名中的四个参数做如下说明：

1. request：要执行的操作类型
2. pid：被追踪的目标进程ID
3. addr：被监控的目标内存地址
4. data：保存读取出或者要写入的数据、

其中 request 的值有：

```c
enum __ptrace_request
{
	PTRACE_TRACEME = 0,		//被调试进程调用
	PTRACE_PEEKDATA = 2,	//查看内存
  	PTRACE_PEEKUSER = 3,	//查看struct user 结构体的值
  	PTRACE_POKEDATA = 5,	//修改内存
  	PTRACE_POKEUSER = 6,	//修改struct user 结构体的值
  	PTRACE_CONT = 7,		//让被调试进程继续
  	PTRACE_SINGLESTEP = 9,	//让被调试进程执行一条汇编指令
  	PTRACE_GETREGS = 12,	//获取一组寄存器(struct user_regs_struct)
  	PTRACE_SETREGS = 13,	//修改一组寄存器(struct user_regs_struct)
  	PTRACE_ATTACH = 16,		//附加到一个进程
  	PTRACE_DETACH = 17,		//解除附加的进程
  	PTRACE_SYSCALL = 24,	//让被调试进程在系统调用前和系统调用后暂停
    ...
};

long int ptrace (enum __ptrace_request __request, ...)
```

得先暂停附加的进程才可以查看或者修改信息！！！

## 查看寄存器
```c
#include <sys/ptrace.h>
#include <sys/wait.h>
#include <sys/reg.h>
#include <sys/user.h>
#include <sys/syscall.h>
#include <stdio.h>
#include <stdint.h>
#include <unistd.h>

int main(int argc,char *argv[])
{
    pid_t pid;
    int orig_rax;
    int iscalling = 0;
    int status = 0;
    uint64_t arg1,arg2,arg3;
    //一组寄存器的值
    struct user_regs_struct regs;

    pid = fork();

    if (pid == 0) { //子进程
        if (ptrace(PTRACE_TRACEME,0,NULL,NULL) < 0) {
            perror("ptrace TRACEME err");
            return -1;
        }

		pid_t pid = getpid();

		if (kill(pid,SIGSTOP) != 0) {
			perror("kill sigstop err");
		}

        write(STDOUT_FILENO,"aaaa -> ",8);
        write(STDOUT_FILENO,"bbbb -> ",8);
        write(STDOUT_FILENO,"cccc -> ",8);

        return 0;
    } else if (pid < 0) {
        perror("fork err");
        return -1;
    }
    //监听子进程的状态
    wait(&status);
    if (WIFEXITED(status))
		return 0;

    //让子进程在调用系统调用时暂停
    if (ptrace(PTRACE_SYSCALL,pid,NULL,NULL) < 0) {
        perror("ptrace SYSCALL err");
        return -1;
    }

    while (1) {
        wait(&status);
        if (WIFEXITED(status))
		    break;

        ptrace(PTRACE_GETREGS,pid,NULL,&regs);//获取寄存器整个结构
  
        orig_rax = regs.orig_rax;//获取系统调用号
        if (orig_rax == SYS_write) {
            if (!iscalling) {//系统调用前
                iscalling = 1;
                arg1 = regs.rdi;
                arg2 = regs.rsi;
                arg3 = regs.rdx;
            } else {//系统调用后
                printf("%lld = write(%ld,\"%s\",%ld)\n",regs.rax,arg1,(char *)arg2,arg3);
                iscalling = 0;
            }
        }
        //让子进程在调用系统调用时暂停
        if (ptrace(PTRACE_SYSCALL,pid,NULL,NULL) < 0) {
            perror("CONT err");
            return -1;
        }
    }

    return 0;
}

```

子进程：

```c
ptrace(PTRACE_TRACEME,0,NULL,NULL);
```

让子进程在系统调用时暂停：

```c
ptrace(PTRACE_SYSCALL,pid,NULL,NULL);
```

获取寄存器结构体

```c
struct user_regs_struct regs;
...
ptrace(PTRACE_GETREGS,pid,NULL,&regs);
orig_rax = regs.orig_rax;
```

PTRACE_GETREGS：获取对应的一组寄存器的值，`user_regs_struct`这个结构体定义在`<sys/user.h>`中，这个结构体保存了一组寄存器的信息：

```c
struct user_regs_struct
{
  __extension__ unsigned long long int r15;
  __extension__ unsigned long long int r14;
  __extension__ unsigned long long int r13;
  __extension__ unsigned long long int r12;
  __extension__ unsigned long long int rbp;
  __extension__ unsigned long long int rbx;
  __extension__ unsigned long long int r11;
  __extension__ unsigned long long int r10;
  __extension__ unsigned long long int r9;
  __extension__ unsigned long long int r8;
  __extension__ unsigned long long int rax;
  __extension__ unsigned long long int rcx;
  __extension__ unsigned long long int rdx;
  __extension__ unsigned long long int rsi;
  __extension__ unsigned long long int rdi;
  __extension__ unsigned long long int orig_rax;
  __extension__ unsigned long long int rip;
  __extension__ unsigned long long int cs;
  __extension__ unsigned long long int eflags;
  __extension__ unsigned long long int rsp;
  __extension__ unsigned long long int ss;
  __extension__ unsigned long long int fs_base;
  __extension__ unsigned long long int gs_base;
  __extension__ unsigned long long int ds;
  __extension__ unsigned long long int es;
  __extension__ unsigned long long int fs;
  __extension__ unsigned long long int gs;
};
```

`rax`：在系统调用完成后，存储系统调用的返回值

`orig_rax`：在系统调用之前，存储系统调用的编号（即原始的系统调用号）

## 内存读取
```c
ptrace(PTRACE_PEEKDATA,pid,&num,NULL);
```

返回值为内存读取得到的值，也就是 rax 的值

## 内存修改
```c
ptrace(PTRACE_POKEDATA,pid,&num,120);
```

## ptrace 绕过 seccomp
在

