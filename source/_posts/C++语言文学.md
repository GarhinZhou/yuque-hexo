---
title: C++语言文学
date: '2024-12-15 23:13:41'
updated: '2025-03-09 00:17:05'
---
## <font style="color:#000000;">三字符组</font>
<font style="color:#000000;">三字符组就是用于表示另一个字符的三个字符序列，又称为三字符序列。三字符序列总是以两个问号开头。</font>

<font style="color:#000000;">三字符序列不太常见，但 C++ 标准允许把某些字符指定为三字符序列。以前为了表示键盘上没有的字符，这是必不可少的一种方法。</font>

<font style="color:#000000;">三字符序列可以出现在任何地方，包括字符串、字符序列、注释和预处理指令。</font>

<font style="color:#000000;">下面列出了最常用的三字符序列：</font>

| <font style="color:#000000;">三字符组</font> | <font style="color:#000000;">替换</font> |
| --- | --- |
| <font style="color:#000000;">??=</font> | <font style="color:#000000;">#</font> |
| <font style="color:#000000;">??/</font> | <font style="color:#000000;">\</font> |
| <font style="color:#000000;">??'</font> | <font style="color:#000000;">^</font> |
| <font style="color:#000000;">??(</font> | <font style="color:#000000;">[</font> |
| <font style="color:#000000;">??)</font> | <font style="color:#000000;">]</font> |
| <font style="color:#000000;">??!</font> | <font style="color:#000000;">|</font> |
| <font style="color:#000000;">??<</font> | <font style="color:#000000;">{</font> |
| <font style="color:#000000;">??></font> | <font style="color:#000000;">}</font> |
| <font style="color:#000000;">??-</font> | <font style="color:#000000;">~</font> |


<font style="color:#000000;">如果希望在源程序中有两个连续的问号，且不希望被预处理器替换，这种情况出现在字符常量、字符串字面值或者是程序注释中，可选办法是用字符串的自动连接："...?""?..."或者转义序列："...?\?..."。</font>

<font style="color:#000000;">从Microsoft Visual C++ 2010版开始，该编译器默认不再自动替换三字符组。如果需要使用三字符组替换（如为了兼容古老的软件代码），需要设置编译器命令行选项/Zc:trigraphs</font>

<font style="color:#000000;">g++仍默认支持三字符组，但会给出编译警告。</font>

## <font style="color:#000000;">基本的内置类型</font>
<font style="color:#000000;">C++ 为程序员提供了种类丰富的内置数据类型和用户自定义的数据类型。下表列出了七种基本的 C++ 数据类型：</font>

| <font style="color:#000000;">类型</font> | <font style="color:#000000;">关键字</font> |
| --- | --- |
| <font style="color:#000000;">布尔型</font> | <font style="color:#000000;">bool</font> |
| <font style="color:#000000;">字符型</font> | <font style="color:#000000;">char</font> |
| <font style="color:#000000;">整型</font> | <font style="color:#000000;">int</font> |
| <font style="color:#000000;">浮点型</font> | <font style="color:#000000;">float</font> |
| <font style="color:#000000;">双浮点型</font> | <font style="color:#000000;">double</font> |
| <font style="color:#000000;">无类型</font> | <font style="color:#000000;">void</font> |
| <font style="color:#000000;">宽字符型</font> | <font style="color:#000000;">wchar_t</font> |


<font style="color:#000000;">其实 wchar_t 是这样来的：</font>

<font style="color:#000000;">typedef short int wchar_t;</font>

<font style="color:#000000;">所以 wchar_t 实际上的空间是和 short int 一样。</font>

<font style="color:#000000;">一些基本类型可以使用一个或多个类型修饰符进行修饰：</font>

+ <font style="color:#000000;">signed</font>
+ <font style="color:#000000;">unsigned</font>
+ <font style="color:#000000;">short</font>
+ <font style="color:#000000;">long</font>

<font style="color:#000000;">下表显示了各种变量类型在内存中存储值时需要占用的内存，以及该类型的变量所能存储的最大值和最小值。</font>

**<font style="color:#000000;">注意：</font>**<font style="color:#000000;">不同系统会有所差异，一字节为 8 位。</font>

**<font style="color:#000000;">注意：</font>**<font style="color:#000000;">默认情况下，int、short、long都是带符号的，即 signed。</font>

**<font style="color:#000000;">注意：</font>**<font style="color:#000000;">long int 8 个字节，int 都是 4 个字节，早期的 C 编译器定义了 long int 占用 4 个字节，int 占用 2 个字节，新版的 C/C++ 标准兼容了早期的这一设定。</font>

| <font style="color:#000000;">类型</font> | <font style="color:#000000;">位</font> | <font style="color:#000000;">范围</font> |
| --- | --- | --- |
| <font style="color:#000000;">char</font> | <font style="color:#000000;">1 个字节</font> | <font style="color:#000000;">-128 到 127 或者 0 到 255</font> |
| <font style="color:#000000;">unsigned char</font> | <font style="color:#000000;">1 个字节</font> | <font style="color:#000000;">0 到 255</font> |
| <font style="color:#000000;">signed char</font> | <font style="color:#000000;">1 个字节</font> | <font style="color:#000000;">-128 到 127</font> |
| <font style="color:#000000;">int</font> | <font style="color:#000000;">4 个字节</font> | <font style="color:#000000;">-2147483648 到 2147483647</font> |
| <font style="color:#000000;">unsigned int</font> | <font style="color:#000000;">4 个字节</font> | <font style="color:#000000;">0 到 4294967295</font> |
| <font style="color:#000000;">signed int</font> | <font style="color:#000000;">4 个字节</font> | <font style="color:#000000;">-2147483648 到 2147483647</font> |
| <font style="color:#000000;">short int</font> | <font style="color:#000000;">2 个字节</font> | <font style="color:#000000;">-32768 到 32767</font> |
| <font style="color:#000000;">unsigned short int</font> | <font style="color:#000000;">2 个字节</font> | <font style="color:#000000;">0 到 65,535</font> |
| <font style="color:#000000;">signed short int</font> | <font style="color:#000000;">2 个字节</font> | <font style="color:#000000;">-32768 到 32767</font> |
| <font style="color:#000000;">long int</font> | <font style="color:#000000;">8 个字节</font> | <font style="color:#000000;">-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807</font> |
| <font style="color:#000000;">signed long int</font> | <font style="color:#000000;">8 个字节</font> | <font style="color:#000000;">-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807</font> |
| <font style="color:#000000;">unsigned long int</font> | <font style="color:#000000;">8 个字节</font> | <font style="color:#000000;">0 到 18,446,744,073,709,551,615</font> |
| <font style="color:#000000;">float</font> | <font style="color:#000000;">4 个字节</font> | <font style="color:#000000;">精度型占4个字节（32位）内存空间，+/- 3.4e +/- 38 (~7 个数字)</font> |
| <font style="color:#000000;">double</font> | <font style="color:#000000;">8 个字节</font> | <font style="color:#000000;">双精度型占8 个字节（64位）内存空间，+/- 1.7e +/- 308 (~15 个数字)</font> |
| <font style="color:#000000;">long long</font> | <font style="color:#000000;">8 个字节</font> | <font style="color:#000000;">双精度型占8 个字节（64位）内存空间，表示 -9,223,372,036,854,775,807 到 9,223,372,036,854,775,807 的范围</font> |
| <font style="color:#000000;">long double</font> | <font style="color:#000000;">16 个字节</font> | <font style="color:#000000;">长双精度型 16 个字节（128位）内存空间，可提供18-19位有效数字。</font> |
| <font style="color:#000000;">wchar_t</font> | <font style="color:#000000;">2 或 4 个字节</font> | <font style="color:#000000;">1 个宽字符</font> |


_<font style="color:#000000;background-color:rgb(243, 247, 240);">注意，各种类型的存储大小与系统位数有关，但目前通用的以64位系统为主。</font>_

_<font style="color:#000000;background-color:rgb(243, 247, 240);">以下列出了32位系统与64位系统的存储大小的差别（windows 相同）：</font>_

![](/images/6828e542d2b1fc3bf15998ec3a6b64cd.jpeg)

<font style="color:#000000;">从上表可得知，变量的大小会根据编译器和所使用的电脑而有所不同。</font>

## <font style="color:#000000;">typedef 声明</font>
<font style="color:#000000;">您可以使用 typedef 为一个已有的类型取一个新的名字。下面是使用 typedef 定义一个新类型的语法：</font>

`<font style="color:#000000;">typedef type newname; </font>`

<font style="color:#000000;">例如，下面的语句会告诉编译器，feet 是 int 的另一个名称：</font>

`<font style="color:#000000;">typedef int feet;</font>`

<font style="color:#000000;">现在，下面的声明是完全合法的，它创建了一个整型变量 distance：</font>

`<font style="color:#000000;">feet distance;</font>`

## <font style="color:#000000;">枚举类型</font>
<font style="color:#000000;">枚举类型(enumeration)是C++中的一种派生数据类型，它是由用户定义的若干枚举常量的集合。</font>

<font style="color:#000000;">如果一个变量只有几种可能的值，可以定义为枚举(enumeration)类型。所谓"枚举"是指将变量的值一一列举出来，变量的值只能在列举出来的值的范围内。</font>

<font style="color:#000000;">创建枚举，需要使用关键字 enum。枚举类型的一般形式为：</font>

```cpp
enum 枚举名{ 
标识符[=整型常数], 
标识符[=整型常数], 
... 
标识符[=整型常数]
} 枚举变量;
```

<font style="color:#000000;">如果枚举没有初始化, 即省掉"=整型常数"时, 则从第一个标识符开始。</font>

<font style="color:#000000;">例如，下面的代码定义了一个颜色枚举，变量 c 的类型为 color。最后，c 被赋值为 "blue"。</font>

```cpp
enum color { red, green, blue } c;
c = blue;
```

<font style="color:#000000;">默认情况下，第一个名称的值为 0，第二个名称的值为 1，第三个名称的值为 2，以此类推。但是，您也可以给名称赋予一个特殊的值，只需要添加一个初始值即可。例如，在下面的枚举中，green 的值为 5。</font>

```cpp
enum color { red, green=5, blue };
```

<font style="color:#000000;">在这里，blue 的值为 6，因为默认情况下，每个名称都会比它前面一个名称大 1，但 red 的值依然为 0。</font>

## <font style="color:#000000;">类型转换</font>
<font style="color:#000000;">类型转换是将一个数据类型的值转换为另一种数据类型的值。</font>

<font style="color:#000000;">C++ 中有四种类型转换：静态转换、动态转换、常量转换和重新解释转换。</font>

### <font style="color:#000000;">静态转换（Static Cast）</font>
<font style="color:#000000;">静态转换是将一种数据类型的值强制转换为另一种数据类型的值。</font>

<font style="color:#000000;">静态转换通常用于比较类型相似的对象之间的转换，例如将 int 类型转换为 float 类型。</font>

<font style="color:#000000;">静态转换不进行任何运行时类型检查，因此可能会导致运行时错误。</font>

<font style="color:#000000;">实例：</font>

```cpp
int i = 10;
float f = static_cast<float>(i); // 静态将int类型转换为float类型
```

### <font style="color:#000000;">动态转换（Dynamic Cast）</font>
<font style="color:#000000;">动态转换通常用于将一个基类指针或引用转换为派生类指针或引用。动态转换在运行时进行类型检查，如果不能进行转换则返回空指针或引发异常。</font>

<font style="color:#000000;">实例：</font>

```cpp
class Base {};
class Derived : public Base {};
Base* ptr_base = new Derived;
Derived* ptr_derived = dynamic_cast<Derived*>(ptr_base); // 将基类指针转换为派生类指针
```

### <font style="color:#000000;">常量转换（Const Cast）</font>
<font style="color:#000000;">常量转换用于将 const 类型的对象转换为非 const 类型的对象。</font>

<font style="color:#000000;">常量转换只能用于转换掉 const 属性，不能改变对象的类型。</font>

<font style="color:#000000;">实例：</font>

```cpp
const int i = 10;
int& r = const_cast<int&>(i); // 常量转换，将const int转换为int
```

### <font style="color:#000000;">重新解释转换（Reinterpret Cast）</font>
<font style="color:#000000;">重新解释转换将一个数据类型的值重新解释为另一个数据类型的值，通常用于在不同的数据类型之间进行转换。</font>

<font style="color:#000000;">重新解释转换不进行任何类型检查，因此可能会导致未定义的行为。</font>

<font style="color:#000000;">实例：</font>

```cpp
int i = 10;
float f = reinterpret_cast<float&>(i); // 重新解释将int类型转换为float类型
```

## <font style="color:#000000;">流插入和流提取操作符（流操作符）</font>
<font style="color:#000000;">在 C++ 中，<< 和 >> 作为 流操作符 用于输入输出（I/O）操作，通常与 std::cin（标准输入）和 std::cout（标准输出）一起使用。</font>

`<font style="color:#000000;"><<</font>`<font style="color:#000000;"> 被称为 流插入操作符（Stream Insertion Operator），用于将数据插入到输出流中，通常用在输出操作中。</font>

`<font style="color:#000000;">>></font>`<font style="color:#000000;"> 被称为 流提取操作符（Stream Extraction Operator），用于从输入流中提取数据，通常用在输入操作中。</font>

<font style="color:#000000;">示例：</font>

```cpp
#include <iostream>
using namespace std;

int main() {
    int x, y;

    // 使用流提取操作符 >> 从输入流中读取数据
    cout << "请输入两个整数：";
    cin >> x >> y;  // 从标准输入读取两个整数

    // 使用流插入操作符 << 向输出流写数据
    cout << "你输入的两个整数是：" << x << " 和 " << y << endl;

    return 0;
}
```

<font style="color:#000000;">在这个例子中：</font>

`<font style="color:#000000;">cin >> x >> y; </font>`<font style="color:#000000;">将从标准输入流（通常是键盘）提取数据并存储到变量 x 和 y 中。</font>

`<font style="color:#000000;">cout << "你输入的两个整数是：" << x << " 和 " << y << endl; </font>`<font style="color:#000000;">将文本和变量输出到标准输出流（通常是屏幕）。</font>

