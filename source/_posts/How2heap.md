---
title: How2heap
date: '2025-09-05 11:41:21'
updated: '2025-12-17 15:01:10'
---
之前学堆的时候没看过这个poc，现在从高版本往回看一看，高版本全没学，估计看得也挺慢的

自己写了一个增删改查的C程序方便以后换libc版本来调试：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void* heap_ptrs[30];
unsigned int heap_sizes[30];

int readint()
{
    char s[8];
    if ( fgets(s, 8, stdin) )
        return atoi(s);
    else
        return -1;
}

void main(){
    setbuf(stdout, NULL);
    setbuf(stdin, NULL);
    setbuf(stderr, NULL);

    puts("Welcome to heap challenge!");
    while(1){
        puts("Choose your option:");
        puts("1. Allocate");
        puts("2. Free");
        puts("3. Edit");
        puts("4. Show");
        puts("5. Exit");
        int index;
        int size;
        switch(readint()){
            case 1:
                while(1){
                    puts("Input index:");
                    scanf("%d", &index);
                    if(heap_sizes[index] == 0) break;
                    else puts("!!!Index occupied");
                }
                puts("Input size:");
                scanf("%d", &size);
                heap_sizes[index] = size;
                getchar();
                puts("Allocate way:");
                switch (readint())
                {
                    case 1:
                        heap_ptrs[index] = malloc(size);
                        if(heap_ptrs[index]) {
                            puts("Malloc success!\n");
                        }
                        break;
                    case 2:
                        heap_ptrs[index] = calloc(1, size);
                        if(heap_ptrs[index]) {
                            puts("Calloc success!\n");
                        }
                        break;
                    default:
                        puts("!!!Invalid way\n");
                        break;
                }
                break;
            case 2:
                puts("Input index:");
                scanf("%d", &index);
                getchar();
                if(heap_ptrs[index] == 0) {
                    puts("!!!Index empty\n");
                    break;
                }
                free(heap_ptrs[index]);
                puts("Free success!\n");
                break;
            case 3:
                int length = 0;
                puts("Input index:");
                scanf("%d", &index);
                getchar();
                if(heap_ptrs[index] == 0) {
                    puts("!!!Index empty\n");
                    break;
                }
                puts("Input length:");
                scanf("%d", &length);
                getchar();
                puts("Input data:");
                read(0, heap_ptrs[index],length);
                puts("Edit success!\n");
                break;
            case 4:
                puts("Input index:");
                scanf("%d", &index);
                getchar();
                if(heap_ptrs[index] == 0) {
                    puts("!!!Index empty\n");
                    break;
                }
                puts("Data:");
                puts((char*)heap_ptrs[index]);
                break;
            case 5:
                exit(0);
            default:
                puts("!!!Invalid option\n");
                break;
       }
    }
}
```

记得把PIE关了，才发现之前gpt给的资料错了：

```bash
gcc -no-pie heap.c -o heap
```

对应的python交互脚本：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=process("./heap")
# link=remote('36.212.14.99',27719)
# elf=ELF("./heap")

gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.39-0ubuntu8.4_amd64/')
pause()

def malloc(index,size):
    link.sendlineafter(b'Choose your option:',b'1')
    link.sendlineafter(b'Input index:',str(index).encode())
    link.sendlineafter(b'Input size:',str(size).encode())
    link.sendlineafter(b'Allocate way:',str(1).encode())

def calloc(index,size):
    link.sendlineafter(b'Choose your option:',b'1')
    link.sendlineafter(b'Input index:',str(index).encode())
    link.sendlineafter(b'Input size:',str(size).encode())
    link.sendlineafter(b'Allocate way:',str(2).encode())

def delete(index):
    link.sendlineafter(b'Choose your option:',b'2')
    link.sendlineafter(b'Input index:',str(index).encode())

def edit(index,length,content):
    link.sendlineafter(b'Choose your option:',b'3')
    link.sendlineafter(b'Input index:',str(index).encode())
    link.sendlineafter(b'Input length:',str(length).encode())
    link.sendafter(b'Input data:',content)

def show(index):
    link.sendlineafter(b'Choose your option:',b'4')
    link.sendlineafter(b'Input index:',str(index).encode())

link.interactive()
```



