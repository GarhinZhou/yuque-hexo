---
title: Linux编程期末报告
date: '2025-06-15 10:40:46'
updated: '2025-06-25 23:00:59'
---
程序名：pwnertools（简称 prts）

## 任务描述
在参与 CTF 竞赛（二进制方向）时经常会在 linux shell 当中有一些对文件的操作以及其他命令行工具的使用，决定利用 linux shell 脚本实现对这些常用操作的整合

## 实现思路
直接执行 shell 脚本可以进入菜单命令行界面，可以查看功能选择执行

通过传入参数也可以直接使用其中功能

个人常用的操作有：

1. 修改二进制文件的权限为可执行
2. 查看二进制 libc 库文件的版本以及本机 libc 版本
3. 利用 checksec 查看二进制可执行文件的保护情况
4. 利用 patchelf 对二进制可执行文件的依赖 libc 进行修改
5. 利用 one_gadget、ROPgadget 工具收集 gadgets、静态一把梭
6. 利用 seccomp-tools 查看可执行文件的 seccomp 规则



用 case 来对应菜单栏的选项和参数选项

标题艺术字用 figlet 实现的，echo 的时候要注意转义字符

## 核心代码片段
```shell
#!/bin/bash

options=("1. Executable"
         "2. Libc version in libc"
         "3. Checksec(Could not run in superuser mode)"
         "4. Patch binary's rpath and interpreter"
         "5. One_gadget in libc"
         "6. ROPgadget in binary"
         "7. Seccomp rules in binary")

file_path=$1
if [[ $# -ge 2 ]]; then #检查参数个数是否大于等于2
    option=$2
    if [ ! -f "$file_path" ]; then
        echo "File not found: $file_path"
        exit 1
    fi
    case "$option" in
        -e)
            chmod +x $file_path
            echo "File is executable."
            ls --color
            exit 0;;
        -l)
            sudo strings $file_path | grep GLIBC
            exit 0;;
        -c)
            checksec $file_path
            exit 0;;
        -p)
            if [[ ! -d $3 ]]; then #检查第三个参数是否为目录
                read -p "Binary's rpath:" rpath
            else
                rpath=$3
            fi
            if [[ ! -f $4 ]]; then #检查第四个参数是否为文件
                read -p "Binary's interpreter:" interpreter
            else
                interpreter=$4
            fi
            if [[ ! -d $rpath ]]; then #检查rpath是否为目录
                echo "Rpath not found: $rpath"
                exit 1
            else
                echo "Rpath found: $rpath"
            fi
            if [[ ! -f $interpreter ]]; then #检查interpreter是否为文件
                echo "Interpreter not found: $interpreter"
                exit 1
            else
                echo "Interpreter found: $interpreter"
            fi
            patchelf --set-rpath $rpath $file_path
            echo "Rpath patched."
            patchelf --set-interpreter $interpreter $file_path
            echo "Interpreter patched."
            exit 0;;
        -o)
            one_gadget $file_path
            exit 0;;
        -r)
            read -p "Assembly Instructions:" assembly_instructions
            read -p "Register:" register
            ROPgadget --binary $file --only $assembly_instructions | grep $register
            exit 0;;
        -s)
            seccomp-tools dump $file_path
            exit 0;;
        *)
            echo "Invalid option. Use $0 -h for help."
            exit 1;;
    esac
fi

#标题
echo -e "\033[36m                               _              _"
echo -e " _ ____      ___ __   ___ _ __| |_ ___   ___ | |___"
echo -e "| '_ \\ \\ /\\ / / '_ \\ / _ \\ '__| __/ _ \\ / _ \\| / __|"
echo -e "| |_) \\ V  V /| | | |  __/ |  | || (_) | (_) | \\__ \\"
echo -e "| .__/ \\_/\\_/ |_| |_|\___|_|   \\__\\___/ \\___/|_|___/"
echo -e "|_|\033[0m"

echo -e "\033[32mOPTIONS:(Ctrl+C to exit. Use prts -h for help)\033[0m"
for option in "${options[@]}"; do
        echo "$option"
done

if [[ ! -f $file_path ]]; then
    if [[ $file_path == 0 ]]; then
        echo "File not found: $file_path"
    elif [[ $file_path == "-h" || $file_path == "-help" ]]; then
        echo -e "\033[31mHelp:\033[0m"
        echo "$0 [file_path] [option]"
        echo "Use -e, -b, -c, -p, -o, -r, -s as options."
        echo "Patch binary's rpath and interpreter can use -p [rpath] [interpreterpath]."
        exit 0
    fi
    echo -e "\033[31mCurrent path:\033[0m" #红色字体
    ls --color #启用颜色输出
    while true; do
        read -p "Enter file(with path):" file
        if [[ ! -f $file ]]; then
            echo "File not found: $file"
            continue
        else
            echo "File found: $file"
            break
        fi
    done
else
    file=$file_path
fi

while true; do
    read -p "Enter choice(number): " choice
    case $choice in
        1)
            chmod +x $file
            echo "File made executable."
            echo -e "\033[31mCurrent path:\033[0m" #红色字体
            ls --color;; #启用颜色输出
        2)
            sudo strings $file | grep GLIBC;;
        3)
            checksec $file;;
        4)
            read -p "Binary's rpath:" rpath 
            read -p "Binary's interpreter:" interpreter
            if [[ ! -f $interpreter ]]; then
                echo "Interpreter not found: $interpreter"
                continue
            else
                echo "Interpreter found: $interpreter"
                break
            fi
            patchelf --set-rpath $rpath $file
            echo "Rpath patched."
            patchelf --set-interpreter $interpreter $file
            echo "Interpreter patched.";;
        5)
            one_gadget $file;;
        6)
            read -p "Assembly Instructions:" assembly_instructions
            read -p "Register:" register
            ROPgadget --binary $file --only $assembly_instructions | grep $register;;
        7)
            seccomp-tools dump $file;;
        *)
            echo "Invalid choice. Please try again.";;
    esac
    echo ""
done
```

## 运行效果截图
### 直接运行
![](/images/48d8bbfd258ac6f5dd986e7b9549db78.png)

### 传入参数执行
![](/images/cc3768d6af8e65ddcf410945b8a3de11.png)

## 总结
利用这个小工具可以一定程度上提高比赛时做题的效率

