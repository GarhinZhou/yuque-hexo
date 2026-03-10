---
title: VM PWN
date: '2025-11-11 15:56:19'
updated: '2025-11-11 15:57:59'
---
## <font style="background-color:rgba(255, 255, 255, 0);">羊城杯 Mvmps</font>
<font style="background-color:rgba(255, 255, 255, 0);">没做过vm题目...</font>

<font style="background-color:rgba(255, 255, 255, 0);">这种题目的逆向难度好大，都不知道如果线下赛出这种题目的得怎么办...</font>

<font style="background-color:rgba(255, 255, 255, 0);">在线的话就交给通义千问了，这个感觉比deepseek好使</font>

<font style="background-color:rgba(255, 255, 255, 0);">✅</font><font style="background-color:rgba(255, 255, 255, 0);"> 类型 0：</font>`<font style="background-color:rgba(255, 255, 255, 0);">sub_401546</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → 控制流 & 内存 & 系统调用</font>

| <font style="background-color:rgba(255, 255, 255, 0);">Opcode >> 2</font> | <font style="background-color:rgba(255, 255, 255, 0);">汇编指令</font> | <font style="background-color:rgba(255, 255, 255, 0);">功能</font> |
| --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">$ (36)</font> | <font style="background-color:rgba(255, 255, 255, 0);">subsp $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">sp -= 4 * imm</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">% (37)</font> | <font style="background-color:rgba(255, 255, 255, 0);">addsp $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">sp += 4 * imm</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">) (41)</font> | <font style="background-color:rgba(255, 255, 255, 0);">jz reg, offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">若 flag==0，则 pc += offset</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">* (42)</font> | <font style="background-color:rgba(255, 255, 255, 0);">jnz reg, offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">若 flag!=0，则 pc += offset</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">43</font> | <font style="background-color:rgba(255, 255, 255, 0);">jl reg, offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">若 flag==1（小于），跳转</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">, (44)</font> | <font style="background-color:rgba(255, 255, 255, 0);">je reg, offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">若 flag==0（等于），跳转</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">- (45)</font> | <font style="background-color:rgba(255, 255, 255, 0);">jg reg, offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">若 flag==2（大于），跳转</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">. (46)</font> | <font style="background-color:rgba(255, 255, 255, 0);">jge reg, offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">若 flag!=1（≥），跳转</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">/ (47)</font> | <font style="background-color:rgba(255, 255, 255, 0);">jmp offset</font> | <font style="background-color:rgba(255, 255, 255, 0);">无条件跳转（pc = pc + offset 或 pc - offset）</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">0 (48)</font> | <font style="background-color:rgba(255, 255, 255, 0);">push reg</font> | <font style="background-color:rgba(255, 255, 255, 0);">sp -= 4; *(base + sp) = reg</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">3 (51)</font> | <font style="background-color:rgba(255, 255, 255, 0);">sys_read(fd, buf, len)</font> | <font style="background-color:rgba(255, 255, 255, 0);">系统调用 read</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">4 (52)</font> | <font style="background-color:rgba(255, 255, 255, 0);">sys_write(fd, buf, len)</font> | <font style="background-color:rgba(255, 255, 255, 0);">系统调用 write，但需 reg0 非空</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">: (58)</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov reg, [reg]</font> | <font style="background-color:rgba(255, 255, 255, 0);">寄存器间内存访问（带边界检查）</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">; (59)</font> | <font style="background-color:rgba(255, 255, 255, 0);">pop reg</font> | <font style="background-color:rgba(255, 255, 255, 0);">reg = *(base + sp); sp += 4 * (imm+1)</font> |


<font style="background-color:rgba(255, 255, 255, 0);">注意：</font>`<font style="background-color:rgba(255, 255, 255, 0);">/</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 的跳转支持符号位判断（bit 23 为符号）：</font>

+ <font style="background-color:rgba(255, 255, 255, 0);">若 </font>`<font style="background-color:rgba(255, 255, 255, 0);">imm & 0x800000</font>`<font style="background-color:rgba(255, 255, 255, 0);">，则 </font>`<font style="background-color:rgba(255, 255, 255, 0);">pc -= imm & 0x7FFFFF</font>`
+ <font style="background-color:rgba(255, 255, 255, 0);">否则 </font>`<font style="background-color:rgba(255, 255, 255, 0);">pc += imm</font>`

---

<font style="background-color:rgba(255, 255, 255, 0);">✅</font><font style="background-color:rgba(255, 255, 255, 0);"> 类型 1：</font>`<font style="background-color:rgba(255, 255, 255, 0);">sub_401C38</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → 寄存器与栈操作</font>

| <font style="background-color:rgba(255, 255, 255, 0);">Opcode >> 2</font> | <font style="background-color:rgba(255, 255, 255, 0);">汇编指令</font> | <font style="background-color:rgba(255, 255, 255, 0);">功能</font> |
| --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">31</font> | <font style="background-color:rgba(255, 255, 255, 0);">push regN</font> | <font style="background-color:rgba(255, 255, 255, 0);">sp -= 4; *(base + sp) = regN （N 由 imm 指定）</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">32</font> | <font style="background-color:rgba(255, 255, 255, 0);">pop regN</font> | <font style="background-color:rgba(255, 255, 255, 0);">regN = *(base + sp); sp += 4</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">33</font> | <font style="background-color:rgba(255, 255, 255, 0);">inc regN</font> | <font style="background-color:rgba(255, 255, 255, 0);">regN++</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">34</font> | <font style="background-color:rgba(255, 255, 255, 0);">dec regN</font> | <font style="background-color:rgba(255, 255, 255, 0);">regN--</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">35</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov sp, regN</font> | <font style="background-color:rgba(255, 255, 255, 0);">regN = sp</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">41~46</font> | <font style="background-color:rgba(255, 255, 255, 0);">cmp reg, flag_cond + 跳转</font> | <font style="background-color:rgba(255, 255, 255, 0);">类似类型 0 的条件跳转逻辑</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">47</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov regN, sp</font> | <font style="background-color:rgba(255, 255, 255, 0);">regN = sp</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">48</font> | <font style="background-color:rgba(255, 255, 255, 0);">push $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">sp -= 4; *(base + sp) = imm; reg = imm</font> |


---

<font style="background-color:rgba(255, 255, 255, 0);">✅</font><font style="background-color:rgba(255, 255, 255, 0);"> 类型 2：</font>`<font style="background-color:rgba(255, 255, 255, 0);">sub_401E52</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → 寄存器间运算与内存访问</font>

| <font style="background-color:rgba(255, 255, 255, 0);">Opcode >> 2</font> | <font style="background-color:rgba(255, 255, 255, 0);">汇编指令</font> | <font style="background-color:rgba(255, 255, 255, 0);">功能</font> |
| --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">1,2</font> | <font style="background-color:rgba(255, 255, 255, 0);">cmp regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">设置 flag：regA < regB ? 1 : (regA > regB ? 2 : 0)</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">3</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA = regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">4</font> | <font style="background-color:rgba(255, 255, 255, 0);">xor regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA ^= regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">5</font> | <font style="background-color:rgba(255, 255, 255, 0);">or regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">`regA</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">6</font> | <font style="background-color:rgba(255, 255, 255, 0);">and regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA &= regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">7</font> | <font style="background-color:rgba(255, 255, 255, 0);">shl regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA <<= regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">8</font> | <font style="background-color:rgba(255, 255, 255, 0);">shr regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA >>= regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">9</font> | <font style="background-color:rgba(255, 255, 255, 0);">xchg regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">交换两个寄存器</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">10</font> | <font style="background-color:rgba(255, 255, 255, 0);">add regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA += regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">11</font> | <font style="background-color:rgba(255, 255, 255, 0);">sub regA, regB</font> | <font style="background-color:rgba(255, 255, 255, 0);">regA -= regB</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">12</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov regA, byte [regB]</font> | <font style="background-color:rgba(255, 255, 255, 0);">从内存加载 byte</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">13</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov regA, word [regB]</font> | <font style="background-color:rgba(255, 255, 255, 0);">加载 word</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">14</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov regA, dword [regB]</font> | <font style="background-color:rgba(255, 255, 255, 0);">加载 dword</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">15</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov byte [regB], regA</font> | <font style="background-color:rgba(255, 255, 255, 0);">存储 byte</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">16</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov word [regB], regA</font> | <font style="background-color:rgba(255, 255, 255, 0);">存储 word</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">17</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov dword [regB], regA</font> | <font style="background-color:rgba(255, 255, 255, 0);">存储 dword</font> |


<font style="background-color:rgba(255, 255, 255, 0);">⚠️</font><font style="background-color:rgba(255, 255, 255, 0);"> 所有内存访问都会调用 </font>`<font style="background-color:rgba(255, 255, 255, 0);">sub_401215(addr)</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 检查地址是否 > 0x1000，若越界则调用 </font>`<font style="background-color:rgba(255, 255, 255, 0);">sub_401239()</font>`<font style="background-color:rgba(255, 255, 255, 0);">（可能是 abort）</font>

---

<font style="background-color:rgba(255, 255, 255, 0);">✅</font><font style="background-color:rgba(255, 255, 255, 0);"> 类型 3：</font>`<font style="background-color:rgba(255, 255, 255, 0);">sub_4024A8</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → 寄存器与立即数操作</font>

| <font style="background-color:rgba(255, 255, 255, 0);">Opcode >> 2</font> | <font style="background-color:rgba(255, 255, 255, 0);">汇编指令</font> | <font style="background-color:rgba(255, 255, 255, 0);">功能</font> |
| --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">1</font> | <font style="background-color:rgba(255, 255, 255, 0);">cmp reg, imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">设置 flag</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">2</font> | <font style="background-color:rgba(255, 255, 255, 0);">cmps reg, imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">有符号比较</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">3</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">立即数赋值</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">4</font> | <font style="background-color:rgba(255, 255, 255, 0);">xor reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">异或立即数</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">5</font> | <font style="background-color:rgba(255, 255, 255, 0);">or reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">或立即数</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">6</font> | <font style="background-color:rgba(255, 255, 255, 0);">and reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">与立即数</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">7</font> | <font style="background-color:rgba(255, 255, 255, 0);">shl reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">左移</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">8</font> | <font style="background-color:rgba(255, 255, 255, 0);">shr reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">右移</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">10</font> | <font style="background-color:rgba(255, 255, 255, 0);">add reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">加立即数</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">11</font> | <font style="background-color:rgba(255, 255, 255, 0);">sub reg, $imm</font> | <font style="background-color:rgba(255, 255, 255, 0);">减立即数</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">12</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov reg, byte [$imm]</font> | <font style="background-color:rgba(255, 255, 255, 0);">从绝对地址加载 byte</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">13</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov reg, word [$imm]</font> | <font style="background-color:rgba(255, 255, 255, 0);">加载 word</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">14</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov reg, dword [$imm]</font> | <font style="background-color:rgba(255, 255, 255, 0);">加载 dword</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">15</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov byte [$imm], reg</font> | <font style="background-color:rgba(255, 255, 255, 0);">存储 byte 到绝对地址</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">16</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov word [$imm], reg</font> | <font style="background-color:rgba(255, 255, 255, 0);">存储 word</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">17</font> | <font style="background-color:rgba(255, 255, 255, 0);">mov dword [$imm], reg</font> | <font style="background-color:rgba(255, 255, 255, 0);">存储 dword</font> |


<font style="background-color:rgba(255, 255, 255, 0);">这题的vm没有内存越界读写的问题，系统调用也只定义了0、1、2分别对应read、write和exit</font>

<font style="background-color:rgba(255, 255, 255, 0);">又因为这题partial RELRO，所以可以考虑将exit函数的got表改成system，然后通过系统调用exit来getshell</font>

