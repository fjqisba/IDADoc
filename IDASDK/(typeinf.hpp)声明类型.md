# 声明类型

IDA内部记录着一个声明类型库，通过IDA的Local Types窗口我们可以看到里面的声明类型。

------

获取IDA内部的声明类型库。

```c++
til_t * get_idati(void);
```

------

解析多条声明，并将它们存储至声明类型库。

```c++
int parse_decls(
        til_t *til,
        const char *input,
        printer_t *printer,
        int hti_flags);
```

第一个参数为要存储解析声明结果的声明类型库。

第二个参数代表声明的文本，如果参数四`hti_flags`为HTI_FIL，则代表包含声明的文件路径。

第三个参数为一个回调函数指针，用于输出解析过程的一些信息。

第四个参数用来设置如何解析声明的一些标志。

这个函数使用频率还是很高的，使用例子如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <typeinf.hpp>

til_t* idati = NULL;
bool idaapi run(size_t)
{
	const char Decls[] = "\
	struct __declspec(align(4)) Tree_nod\
	{\
		Tree_nod* _Left;\
		Tree_nod* _Parent;\
		Tree_nod* _Right;\
		char _Color;\
		char _Isnil;\
		int _Myval; \
	};\
	struct map\
	{\
		Tree_nod* _MyHead;\
		unsigned int _Mysize;\
	};";

	idati = (til_t*)get_idati();
	parse_decls(idati, Decls, NULL,0);

	return true;
}
```

调用函数后，类型声明就存储到了IDA的声明类型库中。

------

导入声明类型至IDB中。相当于将Local Types窗口中的声明拷贝至Structures窗口。

```c++
tid_t import_type (const til_t *til, int idx, const char *name, int flags=0)
```

参数一为要导入声明的声明类型库。

参数二为新类型在列表中的位置，一般填-1就行了，表示加入到列表末尾。

参数三为要导入声明的名称。

参数四可不填。

返回值为拷贝得到的结构体的类型ID。

------

修改指定地址的声明。

该函数首先会解析声明文本，然后调用apply_tinfo函数进行修改。

```c++
bool apply_cdecl(til_t *til, ea_t ea, const char *decl, int flags=0);
```

参数一为声明类型库。

参数二为指定的线性地址。

参数三为声明文本。

参数四为附加参数:

TINFO_GUESSED告诉IDA这是一个模糊的声明。

TINFO_DEFINITE告诉IDA这是一个精确的声明。这会影响到IDA的一些解析。

TINFO_DELAYFUNC，如果声明的种类为函数声明，但是指定的线性地址不存在函数，则尝试建立函数。

TINFO_STRICT，在修改声明之前绝不对类型进行转换。

`apply_cdecl`常常被用来修改函数声明，示例如下:

```c++
til_t* idati = NULL;
bool idaapi run(size_t)
{
	idati = (til_t*)get_idati();
	apply_cdecl(idati, 0x401000, "int __cdecl sub_401000(int a, int b, int *c);", TINFO_DEFINITE);	//注意声明末尾有一个;号

	return true;
}
```

