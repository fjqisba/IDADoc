# 公共函数

<pro.h>是IDA工程中被包含的第一个头文件。

此接口定义了最通用的一些类型，函数和数据。



内存相关函数。

```c++
//malloc
void *	qalloc (size_t size);

//realloc
void *	qrealloc (void *alloc, size_t newsize);

//calloc
void *	qcalloc (size_t nitems, size_t itemsize);

//free
void 	qfree (void *alloc);
```

------

字符串相关函数，没啥介绍的，无非和C语言相比多做了一些安全检查。

```c++
char *	qstrncpy (char *dst, const char *src, size_t dstsize);
char *	qstpncpy (char *dst, const char *src, size_t dstsize);
char *	qstrncat (char *dst, const char *src, size_t dstsize);
char *	qstrtok (char *s, const char *delim, char **save_ptr);
int 	qsnprintf (char *buffer, size_t n, const char *format,...);
int 	qsscanf (const char *input, const char *format,...);
```

------

IDA还重新实现了STL模板库里面的一些类，例如

qstring、qvector、qlist、qstack，大多数情况下是可以替代STL库的，在无法满足我们需求的情况下可以使用STL库，例如IDA没有提供std::map模板类。

------

改变文本字符串的编码，函数为change_codepage

```c++
bool change_codepage(qstring *out,const char *in,int incp,int outcp);
```

参数一out为返回的文本内容结果

参数二in为输入的文本内容

参数三incp为输入的文本的编码

参数四outcp为输出的文本的编码

返回值表示转换是否成功。

IDA特意为我们封装了一个ASCII转换为UTF8的函数，如下

```c++
bool acp_utf8(qstring *out, const char *in)
{
  return change_codepage(out, in, CP_ACP, CP_UTF8);
}
```

因为在IDA中用于显示在界面上的文本是基于UTF8的，因此如果我们想要在IDA的界面上显示中文，需使用此函数将ASCII文本转换为UTF8编码。

------

