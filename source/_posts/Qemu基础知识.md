---
title: Qemu基础知识
date: '2025-05-06 14:16:06'
updated: '2025-05-09 09:58:29'
---
MMIO (Memory-Mapped I/O) 和 PMIO (Port-Mapped I/O) 都是计算机中输入输出设备与处理器进行交互的两种方式。

## MMIO (Memory-Mapped I/O)
+ MMIO 是将 I/O 设备的控制寄存器、数据缓冲区等映射到处理器的内存地址空间中的一种方式
+ 通过 MMIO，程序可以像访问内存一样读取或写入 I/O 设备
+ 一般来说，MMIO 可以让 CPU 直接访问硬件设备的寄存器，而不需要特定的 I/O 指令

它有两种类型：

1. 系统内存映射 I/O (System MMIO)：设备的地址与系统内存地址空间共享
2. 专用内存映射 I/O (Dedicated MMIO)：设备的地址与特定的 I/O 空间相关联

## PMIO (Port-Mapped I/O)
+ PMIO 是另一种访问 I/O 设备的方式，在这种方式下，I/O 设备通过特定的 I/O 端口与处理器进行通信
+ 处理器使用 IN 和 OUT 指令来访问 I/O 设备的端口
+ PMIO 通常有较严格的硬件限制，因为它不使用内存地址空间
+ PMIO 访问通常比 MMIO 慢，因为它依赖于特殊的指令

总的来说，MMIO 直接将 I/O 设备映射到内存地址空间，而 PMIO 通过专用端口进行访问MMIO 在现代计算机中更常见，因为它的访问方式更灵活，性能通常更好

