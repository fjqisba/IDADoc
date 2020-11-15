# 反汇编字节相关

在bytes.hpp中提供的接口代表着IDA对二进制代码反汇编字节分析的一些结果。



最常用的就是数据读写了。例如

**get_byte**、**get_word**、**get_dword**等函数，用于获取指定地址的数据。其中**get_bytes**函数用来获取指定大小的连续数据。

------

**patch_byte**、**patch_word**、**patch_dword**等函数，用于修改指定地址的数据。

------

**set_cmt**/**append_cmt**/**get_cmt**函数，用于反汇编窗口添加注释或者获取注释内容。

------

**bin_search2**函数，用于二进制搜索，有两种调用方法，两种方法其实本质是一样的，官方推荐的做法可能是第一种文本模板，文本语法与IDA官方的Alt+B功能相似。

```c++
ea_t bin_search2(ea_t start_ea,ea_t end_ea,const compiled_binpat_vec_t &data,int flags);

ea_t bin_search2(ea_t start_ea,ea_t end_ea,const uchar *image,const uchar *mask,size_t len,int flags);
```

使用示例如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <bytes.hpp>

bool idaapi run(size_t)
{
	compiled_binpat_vec_t binPat;
	parse_binpat_str(&binPat, 0x0, "55 8B EC", 16);

	ea_t SearchStartAddr = 0x401000;
	while (true)
	{
		SearchStartAddr= bin_search2(SearchStartAddr, 0x500000, binPat, 0x0);
		if (SearchStartAddr == BADADDR)
		{
			break;
		}
		msg("[SearchResult]:%a\n", SearchStartAddr);
		SearchStartAddr = SearchStartAddr + 3;
	}

	return true;
}
```

------

