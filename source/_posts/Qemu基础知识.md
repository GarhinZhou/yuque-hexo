---
title: Qemu基础知识
date: '2025-05-06 14:16:06'
updated: '2025-07-25 18:54:47'
---
## 内存映射
## ![](/images/42a0f18d5fb0a7e602428e83566ba874.png)
```arkts
Guest' processes
                     +--------------------+
Virtual addr space   |                    |
                     +--------------------+
                     |                    |
                     \__   Page Table     \__
                        \                    \
                         |                    |  Guest kernel
                    +----+--------------------+----------------+
Guest's phy. memory |    |                    |                |
                    +----+--------------------+----------------+
                    |                                          |
                    \__                                        \__
                       \                                          \
                        |             QEMU process                 |
                   +----+------------------------------------------+
Virtual addr space |    |                                          |
                   +----+------------------------------------------+
                   |                                               |
                    \__                Page Table                   \__
                       \                                               \
                        |                                               |
                   +----+-----------------------------------------------++
Physical memory    |    |                                               ||
                   +----+-----------------------------------------------++
```

## PCI 设备地址空间
PCI设备都有一个配置空间（PCI Configuration Space，在 PCI 设备上），其记录了关于此设备的详细信息。大小为256字节，其中头部64字节是PCI标准规定的，当然并非所有的项都必须填充，位置是固定了，没有用到可以填充0。前16个字节的格式是一定的，包含头部的类型、设备的总类、设备的性质以及制造商等，格式如下：

![](/images/f74de7d32ab9106bb1c25d45f50475e5.png)

比较关键的是其6个BAR(Base Address Registers)，BAR记录了设备所需要的地址空间的类型，基址以及其他属性。BAR的格式如下：

![](/images/2b78abb35663d30ffa6d6f36a8c999f2.png)

设备可以申请两类地址空间，memory space和I/O space，它们用BAR的最后一位区别开来

通过memory space访问设备I/O的方式称为memory mapped I/O，即MMIO，这种情况下，CPU直接使用普通访存指令即可访问设备I/O

通过I/O space访问设备I/O的方式称为port I/O，或者port mapped I/O，这种情况下CPU需要使用专门的I/O指令如IN/OUT访问I/O端口

## MMIO 和 PMIO
MMIO (Memory-Mapped I/O) 和 PMIO (Port-Mapped I/O) 都是计算机中输入输出设备与处理器进行交互的两种方式。

### MMIO (Memory-Mapped I/O)
+ MMIO 是将 I/O 设备的控制寄存器、数据缓冲区等映射到处理器的内存地址空间中的一种方式
+ 通过 MMIO，程序可以像访问内存一样读取或写入 I/O 设备
+ 一般来说，MMIO 可以让 CPU 直接访问硬件设备的寄存器，而不需要特定的 I/O 指令

它有两种类型：

1. 系统内存映射 I/O (System MMIO)：设备的地址与系统内存地址空间共享
2. 专用内存映射 I/O (Dedicated MMIO)：设备的地址与特定的 I/O 空间相关联

### PMIO (Port-Mapped I/O)
+ PMIO 是另一种访问 I/O 设备的方式，在这种方式下，I/O 设备通过特定的 I/O 端口与处理器进行通信
+ 处理器使用 IN 和 OUT 指令来访问 I/O 设备的端口
+ PMIO 通常有较严格的硬件限制，因为它不使用内存地址空间
+ PMIO 访问通常比 MMIO 慢，因为它依赖于特殊的指令

总的来说，MMIO 直接将 I/O 设备映射到内存地址空间，而 PMIO 通过专用端口进行访问MMIO 在现代计算机中更常见，因为它的访问方式更灵活，性能通常更好



## qemu 中查看 pci 设备信息
总之每个 PCI 设备有一个总线号, 一个设备号, 一个功能号标识（`VendorIDs`、`DeviceIDs`、以及`Class Codes`字段）。PCI 规范允许单个系统占用多达 256 个总线, 但是因为 256 个总线对许多大系统是不够的, Linux 现在支持 PCI 域。每个 PCI 域可以占用多达 256 个总线. 每个总线占用 32 个设备, 每个设备可以是 一个多功能卡(例如一个声音设备, 带有一个附加的 CD-ROM 驱动)有最多 8 个功能

```shell
lspci
```

![](/images/eb659cbe98a577d877d33b6f7e9a693d.png)

```shell
lspci -v -m -n -s [设备ID]
```

```shell
lspci -v -s [设备ID] -x
```

![](/images/690044c4d7bda8c414718123d4c700af.png)

查看 pci 设备的内存：

```shell
cat /sys/bus/pci/devices/[设备ID]/resource
```

![](/images/576254345b374341d5da49663b595079.png)

直接看所有 PCI 设备的详细信息：

```shell
lspci -vvv
```

## qemu 中访问 I/O 空间
### 访问 mmio
内核态：

```c
#include <asm/io.h>
#include <linux/ioport.h>

long addr=ioremap(ioaddr,iomemsize);
readb(addr);
readw(addr);
readl(addr);
readq(addr);//qwords=8 btyes

writeb(val,addr);
writew(val,addr);
writel(val,addr);
writeq(val,addr);
iounmap(addr);
```

用户态：通过映射 resource 文件实现内存的访问

```c
#include <assert.h>
#include <fcntl.h>
#include <inttypes.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <unistd.h>
#include<sys/io.h>


unsigned char* mmio_mem;

void die(const char* msg)
{
    perror(msg);
    exit(-1);
}



void mmio_write(uint32_t addr, uint32_t value)
{
    *((uint32_t*)(mmio_mem + addr)) = value;
}

uint32_t mmio_read(uint32_t addr)
{
    return *((uint32_t*)(mmio_mem + addr));
}




int main(int argc, char *argv[])
{

    // Open and map I/O memory for the strng device
    int mmio_fd = open("/sys/devices/pci/0000:00:04.0/resource", O_RDWR | O_SYNC);
    if (mmio_fd == -1)
        die("mmio_fd open failed");

    mmio_mem = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED, mmio_fd, 0);
    if (mmio_mem == MAP_FAILED)
        die("mmap mmio_mem failed");

    printf("mmio_mem @ %p\n", mmio_mem);

    mmio_read(0x128);
    mmio_write(0x128, 1337);

}
```

### 访问 pmio
内核态：

```c
#include <asm/io.h> 
#include <linux/ioport.h>

inb(port);  //读取一字节
inw(port);  //读取两字节
inl(port);  //读取四字节

outb(val,port); //写一字节
outw(val,port); //写两字节
outl(val,port); //写四字节
```

用户态：需要先调用`iopl`函数申请访问端口

```c
#include <sys/io.h>

iopl(3); 
inb(port); 
inw(port); 
inl(port);

outb(val,port); 
outw(val,port); 
outl(val,port);
```

## QOM 编程模型
QEMU提供了一套面向对象编程的模型——QOM（QEMU Object Module），几乎所有的设备如CPU、内存、总线等都是利用这一面向对象的模型来实现的

有几个比较关键的结构体，`TypeInfo`、`TypeImpl`、`ObjectClass`以及`Object`。其中`ObjectClass`、`Object`、`TypeInfo`定义在`include/qom/object.h`中，`TypeImpl`定义在`qom/object.c`中

`TypeInfo`是用户用来定义一个Type的数据结构，用户定义了一个`TypeInfo`，然后调用`type_register`(TypeInfo )或者`type_register_static`(TypeInfo )函数，就会生成相应的`TypeImpl`实例，将这个`TypeInfo`注册到全局的`TypeImpl`的hash表中

> 在 QOM 中，`type` 表示对象的类别，它与对象的类相关。例如，每个设备、每个模拟的硬件都可以视为一个“对象”，并且这些对象都有一个 `type` 来标识其类别。这些 `type` 字符串通常与 QEMU 中的设备类、子系统或模拟的硬件相关联
>

```c
struct TypeInfo
{
    const char *name;
    const char *parent;
    size_t instance_size;
    void (*instance_init)(Object *obj);
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj);
    bool abstract;
    size_t class_size;
    void (*class_init)(ObjectClass *klass, void *data);
    void (*class_base_init)(ObjectClass *klass, void *data);
    void (*class_finalize)(ObjectClass *klass, void *data);
    void *class_data;
    InterfaceInfo *interfaces;
};
```

