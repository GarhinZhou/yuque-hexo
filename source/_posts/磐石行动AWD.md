---
title: 磐石行动AWD
date: '2025-09-05 10:54:11'
updated: '2025-09-20 17:23:50'
---
一共四台机子，两台pwn...

## TrickOrTreat
初始化的函数：

```c
unsigned int init_buffer()
{
    setvbuf(stdin, 0LL, 2, 0LL);
    setvbuf(stdout, 0LL, 2, 0LL);
    setvbuf(stderr, 0LL, 2, 0LL);
    secret = (__int64 (*)(void))mmap((void *)0x111000, 0x1000uLL, 7, 34, -1, 0LL);
    return alarm(0x3E8u);
}
```

这里面申请了一段可执行的内存，看了一眼也没用上，直接把权限patch成可读可写

treat函数：

```c
_DWORD *__fastcall treat()
{
    int v0; // ecx
    _DWORD *result; // rax
    int v2; // [rsp+8h] [rbp-8h]

    v2 = readint();
    puts("Which kind of candy do you want to give to momo?");
    printf("%s?", &candyList[24 * v2]);
    printf("How many?");
    v0 = dword_40B0[6 * v2] - readint();
    result = dword_40B0;
    dword_40B0[6 * v2] = v0;
    return result;
}
```

这里面有数组越界，可以泄露出libc地址，还可以改4字节，但是没法改got表

trick函数：

```c
buf = 0LL;
v20 = 0LL;
v21 = 0LL;
v22 = 0LL;
v23 = 0LL;
v24 = 0LL;
v25 = 0LL;
v26 = 0LL;
v27 = 0LL;
v28 = 0LL;
v29 = 0LL;
v30 = 0LL;
v31 = 0LL;
v32 = 0LL;
v33 = 0LL;
v34 = 0LL;
v35 = 0LL;
v36 = 0LL;
v37 = 0LL;
v38 = 0LL;
v39 = 0LL;
v40 = 0LL;
v41 = 0LL;
v42 = 0LL;
v43 = 0LL;
v44 = 0LL;
v45 = 0LL;
v46 = 0LL;
v47 = 0LL;
v48 = 0LL;
v49 = 0LL;
v50 = 0LL;
puts("Emmmmmm...... Prank Time!");
read(0, &buf, 0x100uLL);
for ( i = 0; i < strlen((const char *)&buf); ++i )
{
    if ( (*((char *)&buf + i) <= 47 || *((char *)&buf + i) > 57)
    && (*((char *)&buf + i) <= 64 || *((char *)&buf + i) > 90)
    && (*((char *)&buf + i) <= 96 || *((char *)&buf + i) > 122) )
    {
        printf("nauty!%c\n", (unsigned int)*((char *)&buf + i));
        return __readfsqword(0x28u) ^ v51;
    }
}
v0 = secret;
v1 = v20;
*(_QWORD *)secret = buf;
*((_QWORD *)v0 + 1) = v1;
v2 = v22;
*((_QWORD *)v0 + 2) = v21;
*((_QWORD *)v0 + 3) = v2;
v3 = v24;
*((_QWORD *)v0 + 4) = v23;
*((_QWORD *)v0 + 5) = v3;
v4 = v26;
*((_QWORD *)v0 + 6) = v25;
*((_QWORD *)v0 + 7) = v4;
v5 = v28;
*((_QWORD *)v0 + 8) = v27;
*((_QWORD *)v0 + 9) = v5;
v6 = v30;
*((_QWORD *)v0 + 10) = v29;
*((_QWORD *)v0 + 11) = v6;
v7 = v32;
*((_QWORD *)v0 + 12) = v31;
*((_QWORD *)v0 + 13) = v7;
v8 = v34;
*((_QWORD *)v0 + 14) = v33;
*((_QWORD *)v0 + 15) = v8;
v9 = v36;
*((_QWORD *)v0 + 16) = v35;
*((_QWORD *)v0 + 17) = v9;
v10 = v38;
*((_QWORD *)v0 + 18) = v37;
*((_QWORD *)v0 + 19) = v10;
v11 = v40;
*((_QWORD *)v0 + 20) = v39;
*((_QWORD *)v0 + 21) = v11;
v12 = v42;
*((_QWORD *)v0 + 22) = v41;
*((_QWORD *)v0 + 23) = v12;
v13 = v44;
*((_QWORD *)v0 + 24) = v43;
*((_QWORD *)v0 + 25) = v13;
v14 = v46;
*((_QWORD *)v0 + 26) = v45;
*((_QWORD *)v0 + 27) = v14;
v15 = v48;
*((_QWORD *)v0 + 28) = v47;
*((_QWORD *)v0 + 29) = v15;
v16 = v50;
*((_QWORD *)v0 + 30) = v49;
*((_QWORD *)v0 + 31) = v16;
secret();
```

很花，但是有一个call的汇编指令，这种直接call进shellcode的洞为什么会出现在AWD里面...

直接把call指令patch成nop指令算了

treat函数：

```c
_DWORD *__fastcall treat()
{
    int v0; // ecx
    _DWORD *result; // rax
    int v2; // [rsp+8h] [rbp-8h]

    v2 = readint();
    puts("Which kind of candy do you want to give to momo?");
    printf("%s?", &candyList[24 * v2]);
    printf("How many?");
    v0 = dword_40B0[6 * v2] - readint();
    result = dword_40B0;
    dword_40B0[6 * v2] = v0;
    return result;
}
```

这里有个数组越界，可以泄露出libc地址

不过不知道数组越界怎么patch啊...还有整型溢出也是...感觉前面提到的跳转指令也只是在有判断语句的时候可以patch

一个问题：alarm函数不敢动，怕把时间缩太短check不过，想着如果攻击时间比check时间长可以试试缩短这个alarm的时间

另外一题没怎么看...

