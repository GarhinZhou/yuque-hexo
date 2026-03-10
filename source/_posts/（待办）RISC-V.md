---
title: （待办）RISC-V
date: '2025-12-19 17:11:04'
updated: '2025-12-27 17:50:51'
---
[https://zhuanlan.zhihu.com/p/578816042](https://zhuanlan.zhihu.com/p/578816042)

指令查询手册：[https://ai-embedded.com/risc-v/riscv-isa-manual/](https://ai-embedded.com/risc-v/riscv-isa-manual/)

RISC-V 指令有以下特点：

+ 完全开放
+ 指令简单
+ 模块化设计，易于扩展

# 寄存器
| **寄存器** | **ABI 名称** | **说明** |
| :--- | :--- | :--- |
| x0 | zero | 0值寄存器，硬编码为0，写入数据忽略，读取数据为0 |
| x1 | ra | 用于返回地址(return address) |
| x2 | sp | 用于栈指针（stack pointer） |
| x3 | gp | 用于通用指针 (global pointer) |
| x4 | tp | 用于线程指针 （thread pointer） |
| x5 | t0 | 用于存放临时数据或者备用链接寄存器 |
| x6~x7 | t1~t2 | 用于存放临时数据寄存器 |
| x8 | s0/fp | 需要保存的寄存器或者帧指针寄存器 |
| x9 | s1 | 需要保存的寄存器 |
| x10~x11 | a0~a1 | 函数传递参数寄存器或者函数返回值寄存器 |
| x12~x17 | a2~a7 | 函数传递参数寄存器 |
| x18~x27 | s2-s11 | 需要保存的寄存器 |
| x28~x31 | t3~t6 | 用于存放临时数据寄存器 |


# 指令集
整数计算指令

```plain
# 算术指令
add   rd, rs1, rs2    # rd = rs1 + rs2
sub   rd, rs1, rs2    # rd = rs1 - rs2

# 立即数算术
addi  rd, rs1, imm    # rd = rs1 + imm
slti  rd, rs1, imm    # rd = (rs1 < imm) ? 1 : 0

# 逻辑指令
and   rd, rs1, rs2
or    rd, rs1, rs2
xor   rd, rs1, rs2

# 移位指令
sll   rd, rs1, rs2    # 逻辑左移
srl   rd, rs1, rs2    # 逻辑右移
sra   rd, rs1, rs2    # 算术右移
```

加载-存储指令

```plain
# 加载指令
lb    rd, offset(rs1)   # 加载字节（符号扩展）
lbu   rd, offset(rs1)   # 加载字节（无符号扩展）
lh    rd, offset(rs1)   # 加载半字
lhu   rd, offset(rs1)   # 加载半字（无符号）
lw    rd, offset(rs1)   # 加载字

# 存储指令
sb    rs2, offset(rs1)  # 存储字节
sh    rs2, offset(rs1)  # 存储半字
sw    rs2, offset(rs1)  # 存储字
```

控制流指令

```plain
# 无条件跳转
jal   rd, offset        # 跳转并链接（用于函数调用）
jalr  rd, rs1, offset   # 间接跳转

# 条件分支
beq   rs1, rs2, offset  # 相等时跳转
bne   rs1, rs2, offset  # 不相等时跳转
blt   rs1, rs2, offset  # 小于时跳转
bge   rs1, rs2, offset  # 大于等于时跳转
bltu  rs1, rs2, offset  # 无符号小于
bgeu  rs1, rs2, offset  # 无符号大于等于
```

系统指令

```plain
ecall                   # 环境调用（系统调用）
ebreak                  # 断点调试
fence                   # 内存屏障
csrrw  rd, csr, rs1     # 原子读写控制状态寄存器
```

标准扩展

M 扩展 - 乘除法

```plain
# 乘法指令
mul    rd, rs1, rs2     # rd = rs1 * rs2（低32位）
mulh   rd, rs1, rs2     # 高32位（有符号）
mulhu  rd, rs1, rs2     # 高32位（无符号）

# 除法指令
div    rd, rs1, rs2     # 有符号除法
divu   rd, rs1, rs2     # 无符号除法
rem    rd, rs1, rs2     # 有符号取余
remu   rd, rs1, rs2     # 无符号取余
```

A 扩展 - 原子操作

```plain
# 原子内存操作
lr.w      rd, (rs1)       # 加载保留
sc.w      rd, rs2, (rs1)  # 条件存储

# 原子读-修改-写
amoswap.w rd, rs2, (rs1)  # 原子交换
amoadd.w  rd, rs2, (rs1)  # 原子加
amoand.w  rd, rs2, (rs1)  # 原子与
amoxor.w  rd, rs2, (rs1)  # 原子异或
```

F/D 扩展 - 浮点运算

```plain
# 单精度浮点（F扩展）
flw     ft, offset(rs1)   # 加载浮点数
fsw     ft, offset(rs1)   # 存储浮点数
fadd.s  fd, fs1, fs2      # 浮点加法
fmul.s  fd, fs1, fs2      # 浮点乘法

# 双精度浮点（D扩展）
fld     ft, offset(rs1)
fsd     ft, offset(rs1)
fadd.d  fd, fs1, fs2
fmul.d  fd, fs1, fs2
```

C 扩展 - 压缩指令

```plain
# 16位压缩指令（提高代码密度）
c.addi  rd, imm          # 压缩加法立即数
c.li    rd, imm          # 加载立即数
c.lw    rd, offset(rs1)  # 压缩加载
c.sw    rs2, offset(rs1) # 压缩存储
c.j     offset           # 压缩跳转
c.beqz  rs1, offset      # 压缩零跳转
```

