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

qstring、qvector、qlist、qstack，为了IDA的插件稳定性考虑，如果我们需要用到STL库，建议还是使用IDA提供的这些。不过IDA似乎没有提供map。。。。。，

------

