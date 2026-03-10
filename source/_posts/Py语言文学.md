---
title: Py语言文学
date: '2024-12-09 21:09:03'
updated: '2025-09-20 17:23:50'
---
## eval()函数
**eval()** 函数用来执行一个字符串表达式，并返回表达式的值。

**字符串表达式**可以包含变量、函数调用、运算符和其他 Python 语法元素。



eval() 方法的语法:

```python
eval(expression[, globals[, locals]])
```

参数：

+ expression -- 表达式。
+ globals -- 变量作用域，全局命名空间，如果被提供，则必须是一个字典对象。
+ locals -- 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。

eval() 函数将字符串 expression 解析为 Python 表达式，并在指定的命名空间中执行它。

返回值：eval() 函数将字符串转换为相应的对象，并返回表达式的结果。

## ljust()函数
返回一个原字符串左对齐,并使用空格填充至指定长度的新字符串。如果指定的长度小于原字符串的长度则返回原字符串。

语法：

```python
str.ljust(width[,fillchar])
```

+ width--指定字符串长度
+ fillchar--填充字符，默认空格

**<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">Shebang</font>**<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">（也叫做</font>**<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">Hashbang</font>**<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">）是指在 Unix-like 操作系统中，脚本文件的第一行用来指定解释器的语法。它通常是一个以 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">#!</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 开头的特殊标记，后面跟着解释器的路径（如 </font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">/bin/bash</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">、</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);">/usr/bin/python</font>`<font style="color:rgb(36, 41, 47);background-color:rgba(255, 255, 255, 0);"> 等）。这种标记让操作系统知道该如何执行该脚本。</font>

```shell
#!/usr/bin/env python3
```

在 python 脚本最前面加上这句就可以直接用`./xxx.py`的指令在命令行中执行 python 脚本而不用加 python3 前缀了

## 切片操作
```python
[start:stop]
[start:stop:step]

例如：
[::-1]倒序
```

## 类型转换
用%p直接打印出来的字符串的十六进制数->字符串

```python
hex=link.recv()
str=hex.to_bytes(转化后字节数,'little').decode() #默认用utf-8解码
```

## 数组
print()可以直接把整个数组打印出来，用len()就可以获取数组元素个数

.append()可以将元素添加到数组的末尾

## 格式化字符串
1. 百分号 (%) 格式化

```python
name = "Alice"
age = 30
# 使用百分号格式化
print("Name: %s, Age: %d" % (name, age))

pi = 3.141592653589793
# 保留两位小数
print("Pi: %.2f" % pi)
```

1. str.format() 方法

```python
name = "Bob"
age = 25
# 使用 str.format() 格式化
print("Name: {}, Age: {}".format(name, age))

# 位置参数
print("Name: {0}, Age: {1}".format(name, age))

# 关键字参数
print("Name: {name}, Age: {age}".format(name="Charlie", age=35))

pi = 3.141592653589793
# 保留两位小数
print("Pi: {:.2f}".format(pi))
```

1. f-strings (格式化字符串字面量)

```python
name = "Eve"
age = 28
# 使用 f-string 格式化
print(f"Name: {name}, Age: {age}")

pi = 3.141592653589793
# 保留两位小数
print(f"Pi: {pi:.2f}")
```

三种方式的区别：

| 方法 | 语法简单性 | 可读性 | 灵活性 | 版本支持 |
| --- | --- | --- | --- | --- |
| % 格式化 | 较旧，较复杂 | 较差 | 较差 | 所有版本 |
| str.format() | 中等 | 较好 | 很好 | Python 2.7 及以上 |
| f-strings | 最简洁 | 最好 | 最好 | Python 3.6 及以上 |


## requests库
```python
# 导入 requests 包
import requests

# 发送请求
x = requests.get('https://www.runoob.com/')

# 返回网页内容
print(x.text)
```

调用requests请求之后会返回一个response对象，里面包含了具体的响应信息：

```python
print(response.status_code)  # 获取响应状态码
print(response.headers)  # 获取响应头
print(response.content)  # 获取响应内容
```

### requests方法
| **方法** | **描述** |
| --- | --- |
| delete(url, args) | 发送 DELETE 请求到指定 url |
| get(url, params, args) | 发送 GET 请求到指定 url |
| head(url, args) | 发送 HEAD 请求到指定 url |
| patch(url, data, args) | 发送 PATCH 请求到指定 url |
| post(url, data, json, args) | 发送 POST 请求到指定 url |
| put(url, data, args) | 发送 PUT 请求到指定 url |
| request(method, url, args) | 向指定的 url 发送指定的请求方法 |


POST示例：

```python
headers = {'User-Agent': 'Mozilla/5.0'}  # 设置请求头
params = {'key1': 'value1', 'key2': 'value2'}  # 设置查询参数
data = {'username': 'example', 'password': '123456'}  # 设置请求体
response = requests.post('https://www.runoob.com', headers=headers, params=params, data=data)
```

## paho-mqtt 库
```python
sudo apt install python3-paho-mqtt
```

