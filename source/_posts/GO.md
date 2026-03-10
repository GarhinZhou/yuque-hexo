---
title: GO
date: '2025-11-18 19:05:11'
updated: '2025-12-05 12:18:32'
---
# <font style="background-color:rgba(255, 255, 255, 0);">开发环境配置</font>
## linux
<font style="background-color:rgba(255, 255, 255, 0);">先下载安装包：</font>[<font style="background-color:rgba(255, 255, 255, 0);">https://golang.google.cn/dl/</font>](https://golang.google.cn/dl/)

<font style="background-color:rgba(255, 255, 255, 0);">然后解压到</font>`<font style="background-color:rgba(255, 255, 255, 0);">/usr/local</font>`

```bash
sudo tar -C /usr/local -xzf go1.23.3.linux-amd64.tar.gz
```

<font style="background-color:rgba(255, 255, 255, 0);">然后设置环境变量：</font>

```bash
vim ~/.bashrc
#在末尾加上：
export PATH="$PATH:/usr/local/go/bin"
```

<font style="background-color:rgba(255, 255, 255, 0);">生效配置：</font>

```bash
source ~/.bashrc
```

<font style="background-color:rgba(255, 255, 255, 0);">用</font>`<font style="background-color:rgba(255, 255, 255, 0);">go version</font>`<font style="background-color:rgba(255, 255, 255, 0);">查看输出是否成功配置</font>

# <font style="background-color:rgba(255, 255, 255, 0);">编译</font>
<font style="background-color:rgba(255, 255, 255, 0);">直接运行</font>

```bash
go run hello.go
```

<font style="background-color:rgba(255, 255, 255, 0);">编译成可执行文件</font>

```bash
go build hello.go
./hello
```

# <font style="background-color:rgba(255, 255, 255, 0);">语法</font>
<font style="background-color:rgba(255, 255, 255, 0);">基础组成：</font>

1. <font style="background-color:rgba(255, 255, 255, 0);">包声明</font>
2. <font style="background-color:rgba(255, 255, 255, 0);">引入包</font>
3. <font style="background-color:rgba(255, 255, 255, 0);">函数</font>
4. <font style="background-color:rgba(255, 255, 255, 0);">变量</font>
5. <font style="background-color:rgba(255, 255, 255, 0);">语句&表达式</font>
6. <font style="background-color:rgba(255, 255, 255, 0);">注释</font>

```go
package main

import "fmt"

func main() {
    /* 这是我的第一个简单的程序 */
    fmt.Println("Hello, World!")
}
```

## <font style="background-color:rgba(255, 255, 255, 0);">注意</font>
<font style="background-color:rgba(255, 255, 255, 0);">比较特别的是标识符的开头大小写的区分：</font>

<font style="background-color:rgba(255, 255, 255, 0);">当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 protected ）</font>

<font style="background-color:rgba(255, 255, 255, 0);">需要注意的是 </font>`<font style="background-color:rgba(255, 255, 255, 0);">{</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 不能单独放在一行：</font>

```go
package main

import "fmt"

func main()  
{  // 错误，{ 不能在单独的行上
    fmt.Println("Hello, World!")
}
```

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="background-color:rgba(255, 255, 255, 0);">GO 当中一行代表一个语句结束，如果同一行要写下多个语句要用</font>`<font style="background-color:rgba(255, 255, 255, 0);">;</font>`<font style="background-color:rgba(255, 255, 255, 0);">分隔</font>

<font style="background-color:rgba(255, 255, 255, 0);">字符串可以用</font>`<font style="background-color:rgba(255, 255, 255, 0);">+</font>`<font style="background-color:rgba(255, 255, 255, 0);">连接</font>

<font style="background-color:rgba(255, 255, 255, 0);"></font><font style="background-color:rgba(255, 255, 255, 0);">Go 代码中会使用到的 25 个关键字或保留字：</font>

| <font style="background-color:rgba(255, 255, 255, 0);">break</font> | <font style="background-color:rgba(255, 255, 255, 0);">default</font> | <font style="background-color:rgba(255, 255, 255, 0);">func</font> | <font style="background-color:rgba(255, 255, 255, 0);">interface</font> | <font style="background-color:rgba(255, 255, 255, 0);">select</font> |
| --- | --- | --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">case</font> | <font style="background-color:rgba(255, 255, 255, 0);">defer</font> | <font style="background-color:rgba(255, 255, 255, 0);">go</font> | <font style="background-color:rgba(255, 255, 255, 0);">map</font> | <font style="background-color:rgba(255, 255, 255, 0);">struct</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">chan</font> | <font style="background-color:rgba(255, 255, 255, 0);">else</font> | <font style="background-color:rgba(255, 255, 255, 0);">goto</font> | <font style="background-color:rgba(255, 255, 255, 0);">package</font> | <font style="background-color:rgba(255, 255, 255, 0);">switch</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">const</font> | <font style="background-color:rgba(255, 255, 255, 0);">fallthrough</font> | <font style="background-color:rgba(255, 255, 255, 0);">if</font> | <font style="background-color:rgba(255, 255, 255, 0);">range</font> | <font style="background-color:rgba(255, 255, 255, 0);">type</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">continue</font> | <font style="background-color:rgba(255, 255, 255, 0);">for</font> | <font style="background-color:rgba(255, 255, 255, 0);">import</font> | <font style="background-color:rgba(255, 255, 255, 0);">return</font> | <font style="background-color:rgba(255, 255, 255, 0);">var</font> |


<font style="background-color:rgba(255, 255, 255, 0);">除了以上介绍的这些关键字，Go 语言还有 36 个预定义标识符：</font>

| <font style="background-color:rgba(255, 255, 255, 0);">append</font> | <font style="background-color:rgba(255, 255, 255, 0);">bool</font> | <font style="background-color:rgba(255, 255, 255, 0);">byte</font> | <font style="background-color:rgba(255, 255, 255, 0);">cap</font> | <font style="background-color:rgba(255, 255, 255, 0);">close</font> | <font style="background-color:rgba(255, 255, 255, 0);">complex</font> | <font style="background-color:rgba(255, 255, 255, 0);">complex64</font> | <font style="background-color:rgba(255, 255, 255, 0);">complex128</font> | <font style="background-color:rgba(255, 255, 255, 0);">uint16</font> |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">copy</font> | <font style="background-color:rgba(255, 255, 255, 0);">false</font> | <font style="background-color:rgba(255, 255, 255, 0);">float32</font> | <font style="background-color:rgba(255, 255, 255, 0);">float64</font> | <font style="background-color:rgba(255, 255, 255, 0);">imag</font> | <font style="background-color:rgba(255, 255, 255, 0);">int</font> | <font style="background-color:rgba(255, 255, 255, 0);">int8</font> | <font style="background-color:rgba(255, 255, 255, 0);">int16</font> | <font style="background-color:rgba(255, 255, 255, 0);">uint32</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">int32</font> | <font style="background-color:rgba(255, 255, 255, 0);">int64</font> | <font style="background-color:rgba(255, 255, 255, 0);">iota</font> | <font style="background-color:rgba(255, 255, 255, 0);">len</font> | <font style="background-color:rgba(255, 255, 255, 0);">make</font> | <font style="background-color:rgba(255, 255, 255, 0);">new</font> | <font style="background-color:rgba(255, 255, 255, 0);">nil</font> | <font style="background-color:rgba(255, 255, 255, 0);">panic</font> | <font style="background-color:rgba(255, 255, 255, 0);">uint64</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">print</font> | <font style="background-color:rgba(255, 255, 255, 0);">println</font> | <font style="background-color:rgba(255, 255, 255, 0);">real</font> | <font style="background-color:rgba(255, 255, 255, 0);">recover</font> | <font style="background-color:rgba(255, 255, 255, 0);">string</font> | <font style="background-color:rgba(255, 255, 255, 0);">true</font> | <font style="background-color:rgba(255, 255, 255, 0);">uint</font> | <font style="background-color:rgba(255, 255, 255, 0);">uint8</font> | <font style="background-color:rgba(255, 255, 255, 0);">uintptr</font> |


<font style="background-color:rgba(255, 255, 255, 0);">程序一般由关键字、常量、变量、运算符、类型和函数组成</font>

<font style="background-color:rgba(255, 255, 255, 0);">程序中可能会使用到这些分隔符：括号 ()，中括号 [] 和大括号 {}</font>

<font style="background-color:rgba(255, 255, 255, 0);">程序中可能会使用到这些标点符号：</font>**<font style="background-color:rgba(255, 255, 255, 0);">.</font>**<font style="background-color:rgba(255, 255, 255, 0);">、</font>**<font style="background-color:rgba(255, 255, 255, 0);">,</font>**<font style="background-color:rgba(255, 255, 255, 0);">、</font>**<font style="background-color:rgba(255, 255, 255, 0);">;</font>**<font style="background-color:rgba(255, 255, 255, 0);">、</font>**<font style="background-color:rgba(255, 255, 255, 0);">:</font>**<font style="background-color:rgba(255, 255, 255, 0);"> 和 </font>**<font style="background-color:rgba(255, 255, 255, 0);">…</font>**

## <font style="background-color:rgba(255, 255, 255, 0);">格式化字符串</font>
<font style="background-color:rgba(255, 255, 255, 0);">Go 语言中使用 fmt.Sprintf 或 fmt.Printf 格式化字符串并赋值给新串：</font>

+ <font style="background-color:rgba(255, 255, 255, 0);">Sprintf 根据格式化参数生成格式化的字符串并返回该字符串</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Printf 根据格式化参数生成格式化的字符串并写入标准输出</font>

<font style="background-color:rgba(255, 255, 255, 0);">格式化符号:</font>

| <font style="background-color:rgba(255, 255, 255, 0);">格式化符号</font> | <font style="background-color:rgba(255, 255, 255, 0);">描述</font> | <font style="background-color:rgba(255, 255, 255, 0);">示例</font> |
| --- | --- | --- |
| **<font style="background-color:rgba(255, 255, 255, 0);">通用格式</font>** | <font style="background-color:rgba(255, 255, 255, 0);"></font> | <font style="background-color:rgba(255, 255, 255, 0);"></font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">%v</font>` | <font style="background-color:rgba(255, 255, 255, 0);">以默认格式输出变量</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%v", 42)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%+v</font>` | <font style="background-color:rgba(255, 255, 255, 0);">对结构体加字段名的方式输出</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%+v", struct{A int}{A: 42})</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%#v</font>` | <font style="background-color:rgba(255, 255, 255, 0);">以 Go 语法格式化输出</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%#v", map[string]int{"a": 1})</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%T</font>` | <font style="background-color:rgba(255, 255, 255, 0);">输出值的类型</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%T", 42)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%%</font>` | <font style="background-color:rgba(255, 255, 255, 0);">输出百分号</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%%")</font>` |
| **<font style="background-color:rgba(255, 255, 255, 0);">布尔值</font>** | <font style="background-color:rgba(255, 255, 255, 0);"></font> | <font style="background-color:rgba(255, 255, 255, 0);"></font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">%t</font>` | <font style="background-color:rgba(255, 255, 255, 0);">输出</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>`<font style="background-color:rgba(255, 255, 255, 0);">true</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">或</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>`<font style="background-color:rgba(255, 255, 255, 0);">false</font>` | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%t", true)</font>` |
| **<font style="background-color:rgba(255, 255, 255, 0);">整数</font>** | <font style="background-color:rgba(255, 255, 255, 0);"></font> | <font style="background-color:rgba(255, 255, 255, 0);"></font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">%b</font>` | <font style="background-color:rgba(255, 255, 255, 0);">二进制表示</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%b", 5)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%c</font>` | <font style="background-color:rgba(255, 255, 255, 0);">Unicode 对应字符</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%c", 65)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%d</font>` | <font style="background-color:rgba(255, 255, 255, 0);">十进制表示</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%d", 42)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%o</font>` | <font style="background-color:rgba(255, 255, 255, 0);">八进制表示</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%o", 10)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%x</font>` | <font style="background-color:rgba(255, 255, 255, 0);">十六进制表示（小写字母）</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%x", 15)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%X</font>` | <font style="background-color:rgba(255, 255, 255, 0);">十六进制表示（大写字母）</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%X", 15)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%U</font>` | <font style="background-color:rgba(255, 255, 255, 0);">Unicode 格式输出</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%U", 65)</font>` |
| **<font style="background-color:rgba(255, 255, 255, 0);">浮点数</font>** | <font style="background-color:rgba(255, 255, 255, 0);"></font> | <font style="background-color:rgba(255, 255, 255, 0);"></font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">%f</font>` | <font style="background-color:rgba(255, 255, 255, 0);">十进制浮点数</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%f", 3.14)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%e</font>` | <font style="background-color:rgba(255, 255, 255, 0);">科学计数法（小写 e）</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%e", 3.14)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%E</font>` | <font style="background-color:rgba(255, 255, 255, 0);">科学计数法（大写 E）</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%E", 3.14)</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%g</font>` | <font style="background-color:rgba(255, 255, 255, 0);">自动选择</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>`<font style="background-color:rgba(255, 255, 255, 0);">%f</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">或</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>`<font style="background-color:rgba(255, 255, 255, 0);">%e</font>`<br/><font style="background-color:rgba(255, 255, 255, 0);"> </font><font style="background-color:rgba(255, 255, 255, 0);">的简洁表示</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%g", 3.14)</font>` |
| **<font style="background-color:rgba(255, 255, 255, 0);">字符串与字节</font>** | <font style="background-color:rgba(255, 255, 255, 0);"></font> | <font style="background-color:rgba(255, 255, 255, 0);"></font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">%s</font>` | <font style="background-color:rgba(255, 255, 255, 0);">普通字符串</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%s", "Go")</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%q</font>` | <font style="background-color:rgba(255, 255, 255, 0);">带双引号的字符串或字符</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%q", "Go")</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%x</font>` | <font style="background-color:rgba(255, 255, 255, 0);">每个字节用两字符十六进制表示</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%x", "abc")</font>` |
| `<font style="background-color:rgba(255, 255, 255, 0);">%X</font>` | <font style="background-color:rgba(255, 255, 255, 0);">十六进制（大写）表示</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%X", "abc")</font>` |
| **<font style="background-color:rgba(255, 255, 255, 0);">指针</font>** | <font style="background-color:rgba(255, 255, 255, 0);"></font> | <font style="background-color:rgba(255, 255, 255, 0);"></font> |
| `<font style="background-color:rgba(255, 255, 255, 0);">%p</font>` | <font style="background-color:rgba(255, 255, 255, 0);">指针地址</font> | `<font style="background-color:rgba(255, 255, 255, 0);">fmt.Printf("%p", &x)</font>` |


```bash
%[flags][width][.precision]verb
```

+ <font style="background-color:rgba(255, 255, 255, 0);">flags：用于控制格式化输出的标志（可选） </font>
    - <font style="background-color:rgba(255, 255, 255, 0);">-：左对齐</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">+：始终显示数值的符号</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">0：用零填充</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">#：为二进制、八进制、十六进制等加上前缀</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">空格：正数前加空格，负数前加 -</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">width：输出宽度（可选）</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">.precision：浮点数小数点后的位数（可选）</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">verb：用于指定数据的格式化方式</font>

## <font style="background-color:rgba(255, 255, 255, 0);">数据类型</font>
1. 布尔型：布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true
2. 数字类型：整型 int 和浮点型 float32、float64，Go 语言支持整型和浮点型数字，并且支持复数，其中位的运算采用补码
3. 字符串类型：字符串就是一串固定长度的字符连接起来的字符序列。Go 的字符串是由单个字节连接起来的。Go 语言的字符串的字节使用 UTF-8 编码标识 Unicode 文本
4. <font style="background-color:rgba(255, 255, 255, 0);">派生类型：</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">指针类型（Pointer）</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">数组类型</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">结构化类型(struct)</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Channel 类型</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">函数类型</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">切片类型</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">接口类型（interface）</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">Map 类型</font>

### <font style="background-color:rgba(255, 255, 255, 0);">数字类型</font>
<font style="background-color:rgba(255, 255, 255, 0);">Go 也有基于架构的类型，例如：int、uint 和 uintptr</font>

1. <font style="background-color:rgba(255, 255, 255, 0);">uint8：无符号 8 位整型 (0 到 255)</font>
2. <font style="background-color:rgba(255, 255, 255, 0);">uint16：无符号 16 位整型 (0 到 65535)</font>
3. <font style="background-color:rgba(255, 255, 255, 0);">uint32：无符号 32 位整型 (0 到 4294967295)</font>
4. <font style="background-color:rgba(255, 255, 255, 0);">uint64：无符号 64 位整型 (0 到 18446744073709551615)</font>
5. <font style="background-color:rgba(255, 255, 255, 0);">int8：有符号 8 位整型 (-128 到 127)</font>
6. <font style="background-color:rgba(255, 255, 255, 0);">int16：有符号 16 位整型 (-32768 到 32767)</font>
7. <font style="background-color:rgba(255, 255, 255, 0);">int32：有符号 32 位整型 (-2147483648 到 2147483647)</font>
8. <font style="background-color:rgba(255, 255, 255, 0);">int64：有符号 64 位整型 (-9223372036854775808 到 9223372036854775807)</font>

### <font style="background-color:rgba(255, 255, 255, 0);">浮点型</font>
1. float32：IEEE-754 32位浮点型数
2. float64：IEEE-754 64位浮点型数
3. complex64：32 位实数和虚数
4. complex128：64 位实数和虚数

### <font style="background-color:rgba(255, 255, 255, 0);">其他数字类型</font>
1. <font style="background-color:rgba(255, 255, 255, 0);">byte：类似 uint8</font>
2. <font style="background-color:rgba(255, 255, 255, 0);">rune：类似 int32</font>
3. <font style="background-color:rgba(255, 255, 255, 0);">uint：32 或 64 位</font>
4. <font style="background-color:rgba(255, 255, 255, 0);">int：与 uint 一样大小</font>
5. <font style="background-color:rgba(255, 255, 255, 0);">uintptr：无符号整型，用于存放一个指针</font>

## <font style="background-color:rgba(255, 255, 255, 0);">传参寄存器</font>
前6个参数：ax,bx,cx,di,si,r8,r9,r10,r11

返回值：ax

