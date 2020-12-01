# 交叉引用

xref.hpp包括和交叉引用相关的一些函数。



交叉引用分为两种:代码交叉引用和数据交叉引用。

交叉引用会自动进行排序。

------

获取某个全局变量所有的交叉引用地址，示例代码如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <xref.hpp>

bool idaapi run(size_t)
{
	//可以自行输入一个全局变量的地址...
	ea_t GlobalVarAddr = 0x489085;

	//打印所有的代码交叉引用
	ea_t XrefAddr = get_first_cref_to(GlobalVarAddr);
	while (XrefAddr != BADADDR)
	{
		msg("CodeXrefAddr:%a\n", XrefAddr);
		XrefAddr = get_next_cref_to(GlobalVarAddr, XrefAddr);
	}
	
	//打印所有的数据交叉引用
	XrefAddr = get_first_dref_to(GlobalVarAddr);
	while (XrefAddr != BADADDR)
	{
		msg("DataXrefAddr:%a\n", XrefAddr);
		XrefAddr = get_next_dref_to(GlobalVarAddr, XrefAddr);
	}
	return true;
}
```

