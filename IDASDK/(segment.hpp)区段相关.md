# 区段相关

segment.hpp包含用来处理程序区段的一些函数。



遍历程序的区段

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <segment.hpp>

bool idaapi run(size_t)
{
	//获取区段个数
	int segCount = get_segm_qty();
	for (int idx = 0; idx < segCount; ++idx)
	{
		//根据区段索引来获取区段结构体
		segment_t* pSegment = getnseg(idx);
		
		//获取区段名称
		qstring SectionName;
		get_segm_name(&SectionName, pSegment);

		msg("segment:%s from %a to %a\n", SectionName.c_str(), pSegment->start_ea, pSegment->end_ea);
	}
	return true;
}
```

------

获取线性地址所在的区段

```c++
segment_t* getseg(ea_t ea);
```

参数ea为程序中的任何一个线性地址，返回线性地址所在区段的结构体指针。

