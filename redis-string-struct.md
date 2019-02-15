# Redis 源码学习——浅谈 Redis 的字符串实现

## 0x00 结构

Redis 的字符串，本质上还是沿用了 C 语言的字符串，但是还是有一些不同。Redis 的字符串附加了一些和字符串本身相关的信息。我觉得这个设计很巧妙（也许是一个常用的技巧，是我太菜了不知道）。

首先来看 sds 的定义

```c
typedef char *sds;
```

在目前的约定下，一个 `char` 是一个字节，这是开辟内存空间的最小的单位了，方便内存管理（我是这样认为的）。关于字符串的信息，定义在了几个结构体里面。

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

这几个结构体，其实是同一个作用，就是记录了字符串的各种信息，它们的区别就是用来表示不同长度的字符串，或者说用来开辟不同大小的空间。里面的几个成员变量，我认为是这样的作用：

- `len` 是用来记录字符串真实长度的。
- `alloc` 是用来记录开辟的空间的总大小。
- `flag` 用来记录这是个什么长度类型的字符串。
- `buf` 这个字符串真正实体的首地址，和 C 语言的字符串是一样的。

## 0x01 新建

好了，字符串的结构大致上清楚了，接下来就是把它们组合起来了。

```c
sds sdsnewlen(const void *init, size_t initlen) {
    void *sh;

    ...

    int hdrlen = sdsHdrSize(type);
    unsigned char *fp; /* flags pointer. */

    sh = s_malloc(hdrlen+initlen+1);
    
    ...
    
    s = (char*)sh+hdrlen;
    fp = ((unsigned char*)s)-1;
    
    ...

    if (initlen && init)
        memcpy(s, init, initlen);
    s[initlen] = '\0';
    return s;
}
```

这里的部分代码代码展示了 Redis 新建字符串的逻辑。传入的参数 `initlen` 是字符串长度，而 `init` 是字符串的值。这里申请了一块完整的 `sds` 空间，空间的大小是 `sdshdr` + `initlen` + 1 的大小，也就是 sds 头和字符串的总长度，当然也不能忘了最后的 `\0`。`s = (char*)sh+hdrlen;` 确保了 `s` 就像是一个 C 字符串一样，尽管它并不是一个简单的字符串。所以，一顿操作之后，拿 `sdshdr8` 做例子，内存空间就变成这个样子。

+-----+-----+-----+-----+-----+------------+-----+---------------------+
|     |     |     |     |     |            |     |
| len |alloc|flags| buf |     |    ...     |  \0 |
|     |     |     | (s) |     |            |     |
+-----+-----+-----+-----+-----+------------+-----+---------------------+

实际上返回的这个 `s` 就是字符串的首地址。那么其它的元素怎么访问呢？就直接这么访问就行了 `sdshdr8 *sh = s - sdshdrlen;` 就可以访问了。感觉这里的设计超级巧妙！

## 0x03 最后

C 语言真的可以写出花来，看了 Redis 的源码算是稍微领教了= =

> 本文为作者自己读 Redis 源码总结的文章，由于作者的水平限制，难免会有错误，欢迎大家指正，感激不尽。