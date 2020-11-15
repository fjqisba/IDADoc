# 导出函数相关

IDA内部维护着一组entry point数据，其中每个entry point:

- 有一个地址
- 有一个名称
- 可能包含一个序号

导出函数被视为entry point，同时程序的执行入口点和TLS回调函数入口也被视为entry point。

<entry.hpp>中提供了对entry point列表的一些操作。



示例代码如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <entry.hpp>

bool idaapi run(size_t)
{
	//获取entry point个数
	size_t entryCount = get_entry_qty();
	for (unsigned int idx = 0; idx < entryCount; ++idx)
	{
		//根据下标来获取序号
		uval_t order = get_entry_ordinal(idx);

		ea_t FuncAddr = get_entry(order);
		qstring FuncName;
		get_entry_name(&FuncName, order);

		//根据序号与函数地址是否相等来判断是否为导出函数
		if (order == FuncAddr)
		{
			msg("[NotExportFunc]:%s--%a\n", FuncName.c_str(), FuncAddr);
		}
		else
		{
			msg("[ExportFunc]:%s--%a\n", FuncName.c_str(), FuncAddr);
		}
 	}

	return true;
}
```

