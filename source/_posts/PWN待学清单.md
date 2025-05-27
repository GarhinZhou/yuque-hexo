---
title: PWN待学清单
date: '2025-01-24 18:23:11'
updated: '2025-05-11 18:17:12'
---
# PWN Survival Guide
## **ELF 保护机制**
    1. RELRO
    2. Stack Canary
    3. NX
    4. PIE / ASLR
    5. <font style="background-color:rgba(255, 255, 255, 0);">seccomp sandbox</font>
    6. printf Fortify

## **栈系列**
    1. rop
        1. ret2text
        2. ret2syscall
        3. ret2libc
        4. ret2csu
        5. <font style="background-color:#FBDE28;">ret2dlresolve （困难）</font>
    2. 栈溢出
        1. stack pivoting
        2. partial overwrite frame
        3. SROP
        4. <font style="background-color:#FBDE28;">setcontext gadget</font>
        5. <font style="background-color:#FBDE28;">libc2.31 magic gadget</font>
    3. shellcode
        1. AMD64 short shellcode
        2. <font style="background-color:#FBDE28;">i386 <=> amd64</font>
        3. Alphanumeric/Printable shellcode
        4. pwntool shellcraft 自动生成 shellcode
    4. one gadget

## **格式化字符串**
    1. 泄漏数据
    2. 修改数据
    3. 定位 (pwndbg fmtarg)
    4. pwntools fmtstr_payload 自动生成 payload

## **其他**
    1. <font style="background-color:rgba(255, 255, 255, 0);">整数溢出</font>
    2. <font style="background-color:#FBDE28;">条件竞争</font>
    3. <font style="background-color:#FBDE28;">未初始化变量</font>

**👇👇👇👇**** 第一个分水岭 ****👇👇👇👇**** **

---

👆👆👆👆👆👆👆👆👆👆👆👆👆

## **堆系列**
    1. glibc
        1. bin
        2. chunk 结构
        3. malloc() / free() 实现
        4. hook
        5. 漏洞类型
            1. Use After Free
            2. Double Free
            3. Heap overflow
            4. Off-by-null
        6. 利用手法
    2. <font style="background-color:#FBDE28;">musl libc</font>

## **FSOP**
    1. <font style="background-color:#FBDE28;">IO_FILE 结构</font>
    2. <font style="background-color:#FBDE28;">glibc vtable</font>
    3. <font style="background-color:#FBDE28;">house of apple</font>

## **非 x86 架构**
    1. aarch64 / arm
    2. <font style="background-color:#FBDE28;">mips</font>
    3. <font style="background-color:#FBDE28;">risc-v</font>

**👇👇👇👇**** 第二个分水岭 ****👇👇👇👇**** **

---

👆👆👆👆👆👆👆👆👆👆👆👆👆

## **Linux 内核**
    1. <font style="background-color:#FBDE28;">调试/运行环境</font>
    2. <font style="background-color:#FBDE28;">常用 API</font>
    3. <font style="background-color:#FBDE28;">提权方式</font>
    4. <font style="background-color:#FBDE28;">内核保护机制</font>
    5. <font style="background-color:#FBDE28;">内核堆 (sl*b)</font>
    6. <font style="background-color:#FBDE28;">内核 ROP</font>
    7. <font style="background-color:#FBDE28;">用户空间与内核空间的切换 </font>
        1. <font style="background-color:#FBDE28;">swapgs_restore_regs_and_return_to_usermode</font>
        2. <font style="background-color:#FBDE28;">syscall_enter</font>

## **虚拟机****逃逸 (qemu)**
1. <font style="background-color:#FBDE28;">mmio</font>
2. <font style="background-color:#FBDE28;">pmio</font>

[QEMU逃逸](https://bbs.pediy.com/thread-265501.htm)

## 浏览器 / Javascript 引擎
<font style="background-color:#FBDE28;">v8</font>

<font style="background-color:#FBDE28;"></font>

# CTFwiki Pwn 部分
## <font style="color:rgba(0, 0, 0, 0.87);">Linux Platform</font>
    - <font style="color:rgba(0, 0, 0, 0.87);">User Mode</font>
        * <font style="color:rgba(0, 0, 0, 0.87);">Environment</font>
        * <font style="color:rgba(0, 0, 0, 0.87);">Exploitation</font>
            + <font style="color:rgba(0, 0, 0, 0.87);">Stack Overflow</font>
                - <font style="color:rgba(0, 0, 0, 0.87);">x86</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);">栈介绍</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/stack-intro/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);">栈溢出原理</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/stackoverflow-basic/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);">基本 ROP</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/basic-rop/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);">中级 ROP</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/medium-rop/)
                    * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">高级 ROP</font>
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">高级 ROP</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/advanced-rop/advanced-rop/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">ret2dlresolve</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/advanced-rop/ret2dlresolve/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">ret2VDSO</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/advanced-rop/ret2vdso/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);">SROP</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/advanced-rop/srop/)
                    * <font style="color:rgba(0, 0, 0, 0.87);">花式栈溢出</font>
                        + [<font style="color:rgba(0, 0, 0, 0.87);">花式栈溢出技巧</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/x86/fancy-rop/)
                - <font style="color:rgba(0, 0, 0, 0.87);">arm</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);">环境搭建</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/arm/environment/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);">Arm ROP</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/arm/rop/)
                - <font style="color:rgba(0, 0, 0, 0.87);">mips</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);">mips - ROP</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/mips/rop/)
                - <font style="color:rgba(0, 0, 0, 0.87);">risc-v</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);">RISC-V</font>](https://ctf-wiki.org/pwn/linux/user-mode/stackoverflow/risc-v/readme/)
            + <font style="color:rgba(0, 0, 0, 0.87);">Format String</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);">原理介绍</font>](https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-intro/)
                - [<font style="color:rgba(0, 0, 0, 0.87);">利用</font>](https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-exploit/)
                - [<font style="color:rgba(0, 0, 0, 0.87);">例子</font>](https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-example/)
                - [<font style="color:rgba(0, 0, 0, 0.87);">检测</font>](https://ctf-wiki.org/pwn/linux/user-mode/fmtstr/fmtstr-detect/)
            + <font style="color:rgba(0, 0, 0, 0.87);">Heap Exploitation</font>
                - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Ptmalloc2</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">堆利用</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/introduction/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">堆概述</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/heap-overview/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">堆相关数据结构</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/heap-structure/)
                    * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">深入理解 Ptmalloc2</font>
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">深入理解堆的实现</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/overview/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">基础操作</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/basic/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">堆初始化</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/heap-init/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">申请内存块</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/malloc/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">释放内存块</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/free/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">tcache</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/tcache/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">malloc_state 相关函数</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/malloc-state/)
                        + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">测试支持</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/implementation/misc/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">堆溢出</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/heapoverflow-basic/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">堆中的 Off-By-One</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/off-by-one/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Chunk Extend and Overlapping</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/chunk-extend-overlapping/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Unlink</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/unlink/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Use After Free</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/use-after-free/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Fastbin Attack</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/fastbin-attack/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Unsorted Bin Attack</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/unsorted-bin-attack/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Large Bin Attack</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/large-bin-attack/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Tcache attack</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/tcache-attack/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House Of Einherjar</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-einherjar/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House Of Force</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-force/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House of Lore</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-lore/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House of Orange</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-orange/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House of Rabbit</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-rabbit/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House of Roman</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-roman/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">House of Pig</font>](https://ctf-wiki.org/pwn/linux/user-mode/heap/ptmalloc2/house-of-pig/)
                - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Musl-mallocng</font>
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">IO_FILE Exploitation</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">FILE 结构</font>](https://ctf-wiki.org/pwn/linux/user-mode/io-file/introduction/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">伪造 vtable 劫持程序流程</font>](https://ctf-wiki.org/pwn/linux/user-mode/io-file/fake-vtable-exploit/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">FSOP</font>](https://ctf-wiki.org/pwn/linux/user-mode/io-file/fsop/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">glibc 2.24 下 IO_FILE 的利用</font>](https://ctf-wiki.org/pwn/linux/user-mode/io-file/exploit-in-libc2.24/)
            + <font style="color:rgba(0, 0, 0, 0.87);">Integer Overflow</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);">整数溢出</font>](https://ctf-wiki.org/pwn/linux/user-mode/integeroverflow/introduction/)
            + <font style="color:rgba(0, 0, 0, 0.87);">Type Confusion</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);">Type Confusion</font>](https://ctf-wiki.org/pwn/linux/user-mode/type-confusion/readme/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Uninitialized Memory</font>
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Race Condition</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Race Condition</font>](https://ctf-wiki.org/pwn/linux/user-mode/race-condition/introduction/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">题目</font>](https://ctf-wiki.org/pwn/linux/user-mode/race-condition/problem/)
        * <font style="color:rgba(0, 0, 0, 0.87);">Defense</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);">Canary</font>](https://ctf-wiki.org/pwn/linux/user-mode/mitigation/canary/)
        * <font style="color:rgba(0, 0, 0, 0.87);">Summary</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);">获取地址</font>](https://ctf-wiki.org/pwn/linux/user-mode/summary/get-address/)
            + [<font style="color:rgba(0, 0, 0, 0.87);">控制程序执行流</font>](https://ctf-wiki.org/pwn/linux/user-mode/summary/hijack-control-flow/)
            + [<font style="color:rgba(0, 0, 0, 0.87);">shell 获取小结</font>](https://ctf-wiki.org/pwn/linux/user-mode/summary/get-shell/)
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Kernel Mode</font>
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Environment</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/environment/readme/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">内核下载与编译</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/environment/build-kernel/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">编译内核驱动</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/environment/build-kernel-module/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Qemu 模拟环境</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/environment/qemu-emulate/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Real Device</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/environment/real-device/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">System.map</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/environment/System.map/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Basic Knowledge</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">基础知识</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/basic-knowledge/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Aim</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/aim/readme/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Privilege Escalation</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/aim/privilege-escalation/readme/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Change Self</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/aim/privilege-escalation/change-self/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Change Others</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/aim/privilege-escalation/change-others/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Information Disclosure</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/aim/info-leak/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">DoS</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/aim/dos/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Defense</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/readme/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Isolation</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/isolation/readme/)
                - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">User and Kernel</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/isolation/user-kernel/readme/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">用户代码不可执行</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/isolation/user-kernel/user-code-execution/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">用户数据不可访问</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/isolation/user-kernel/user-data-access/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">KPTI - Kernel Page Table Isolation</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/isolation/user-kernel/kpti/)
                - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Inside Kernel</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">内部隔离</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/isolation/inner-kernel/heap-chunk/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Access Control</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/access-control/readme/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">信息泄漏</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/access-control/information-disclosure/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Misc</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/access-control/misc/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Detection</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/detection/readme/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Kernel Stack Canary</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/detection/canary/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Randomization</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Introduction</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/randomization/readme/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">KASLR</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/randomization/kaslr/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">FGKASLR</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/defense/randomization/fgkaslr/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Exploitation</font>
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Returned Oriented Programming</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Kernel ROP</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/rop/rop/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">ret2usr（已过时）</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/rop/ret2usr/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">bypass-smep</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/rop/bypass-smep/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">利用 pt_regs 构造通用内核 ROP</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/rop/ret2ptregs/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">ret2dir</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/rop/ret2dir/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Heap Exploitation</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">内核堆概述</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/heap/heap_overview/)
                - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">slub allocator</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">kernel UAF</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/heap/slub/uaf/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Heap Spray</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/heap/slub/spray/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">freelist 劫持</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/heap/slub/freelist/)
                - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Buddy System</font>
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Cross-Cache Overflow & Page-level Heap Fengshui</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/heap/buddy/cross-cache/)
                    * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Page-level UAF</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/heap/buddy/page-uaf/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Race Condition</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Double Fetch</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/race/double-fetch/)
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">userfaultfd 的使用</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/race/userfaultfd/)
            + <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Tricks</font>
                - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">在内存中直接搜索 flag</font>](https://ctf-wiki.org/pwn/linux/kernel-mode/exploitation/tricks/initramfs/)

## <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Windows Platform</font>
    - [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">概述</font>](https://ctf-wiki.org/pwn/windows/readme/)
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">User Mode</font>
        * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Readme</font>](https://ctf-wiki.org/pwn/windows/user-mode/readme/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">栈溢出</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">栈介绍</font>](https://ctf-wiki.org/pwn/windows/user-mode/stackoverflow/stack-introduction/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">栈溢出原理</font>](https://ctf-wiki.org/pwn/windows/user-mode/stackoverflow/stackoverflow-basic/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">执行 Shellcode</font>](https://ctf-wiki.org/pwn/windows/user-mode/stackoverflow/shellcode-in-stack/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">ret2dll 利用</font>](https://ctf-wiki.org/pwn/windows/user-mode/stackoverflow/ret2dll/)
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Kernel Mode</font>

## <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">MacOS Platform</font>
## <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Misc OS Platform</font>
## <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Sandbox Escape</font>
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">python</font>
        * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Python 沙盒</font>](https://ctf-wiki.org/pwn/sandbox/python/python-sandbox-escape/)
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">shell</font>
        * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Shell Sandbox</font>](https://ctf-wiki.org/pwn/sandbox/shell/readme/)
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">seccomp</font>
        * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">C 沙盒逃逸</font>](https://ctf-wiki.org/pwn/sandbox/seccomp/c-sandbox-escape/)
    - <font style="color:rgba(0, 0, 0, 0.87);">namespace</font>
    - <font style="color:rgba(0, 0, 0, 0.87);">chroot</font>
    - <font style="color:rgba(0, 0, 0, 0.87);">docker</font>

## <font style="color:rgba(0, 0, 0, 0.87);">Virtualization</font>
    - <font style="color:rgba(0, 0, 0, 0.87);">基础知识</font>
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">QEMU</font>
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">基础知识</font>
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">环境搭建</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">QEMU 下载与编译</font>](https://ctf-wiki.org/pwn/virtualization/qemu/environment/build-qemu/)
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">编写 QEMU 模拟设备</font>](https://ctf-wiki.org/pwn/virtualization/qemu/environment/build-qemu-dev/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Exploitation</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">QEMU 逃逸入门</font>](https://ctf-wiki.org/pwn/virtualization/qemu/exploitation/intro/)
    - <font style="color:rgba(0, 0, 0, 0.87);">Virtual Box</font>
    - <font style="color:rgba(0, 0, 0, 0.87);">VMWare</font>
    - <font style="color:rgba(0, 0, 0, 0.87);">Parallels</font>

## <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Browser</font>
## <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">Hardware</font>
    - <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">CPU</font>
        * [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">简介</font>](https://ctf-wiki.org/pwn/hardware/cpu/readme/)
        * <font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">侧信道攻击</font>
            + [<font style="color:rgba(0, 0, 0, 0.87);background-color:#FBDE28;">prefetch side-channel attack</font>](https://ctf-wiki.org/pwn/hardware/cpu/side-channel/prefetch/)
    - <font style="background-color:#FBDE28;">可信计算</font>

