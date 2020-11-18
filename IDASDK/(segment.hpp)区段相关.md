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
	int segCount = get_segm_qty();
	for (int idx = 0; idx < segCount; ++idx)
	{
		segment_t* pSegment = getnseg(idx);
		
		qstring SectionName;
		get_segm_name(&SectionName, pSegment);

		msg("segment:%s from %a to %a\n", SectionName.c_str(), pSegment->start_ea, pSegment->end_ea);
	}
	return true;
}
```

