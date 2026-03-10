---
title: fmt家族
date: '2025-11-02 15:43:20'
updated: '2025-12-28 08:48:22'
---
# 输出函数
`printf`、`fprintf`、`dprintf`、`sprintf`、`snprintf`

## fprintf
```c
fprintf(FILE *stream, const char *format, ...);
```

相比 printf ，可以指定格式化输出的文件流

## dprintf
```c
dprintf(int fd, const char *format, ...);
```

相比 printf，可以指定将格式化数据写入文件描述符对应文件

## sprintf
```c
sprintf(char *str, const char *format, ...);
```

相比 printf，可以指定将格式化数据写入对应字符串缓冲区

`snprintf`

```c
snprintf(char *str, size_t size, const char *format, ...);
```

多了个 size 防止缓冲区溢出

# 输入函数
`scanf`、`fscanf`、`sscanf`

scanf("%d")只输入-或+号可以即不是非法又可以不修改到原有数

## fscanf
```c
fscanf(FILE *stream, const char *format, ...);
```

指定从对应文件流读取格式化数据

## sscanf
```c
sscanf(const char *str, const char *format, ...);
```

从对应的缓冲区读取格式化数据

