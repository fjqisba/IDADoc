# IDA自动分析器

在auto.hpp中，包含了关于IDA自动分析引擎相关的一些函数。

当加载一个新的二进制文件的时候，IDA的自动分析引擎便会开始工作。

IDA的自动分析器包含多个分析队列，每个队列有各自的优先级。当所有的分析队列都为空的时候IDA就会结束自动分析。



通过头文件中提供的接口，我们可以对自动分析器进行一些控制。此接口一般来说用的比较少。。。。。。

#### 获取自动分析引擎状态

有的时候我们需要对自动分析引擎的状态进行判断，因为自动分析引擎的结果可能会影响到插件的使用。这个时候可以使用**auto_is_ok**这个函数。如下代码:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <auto.hpp>

bool idaapi run(size_t)
{
	if (!auto_is_ok() && ask_yn(0, "The autoanalysis has not finished yet.\nDo you want to continue?") < 1)
	{
		return false;
	}

	//To do...执行功能
	msg("[IDADemo]:Test\n");
	return true;
}
```



