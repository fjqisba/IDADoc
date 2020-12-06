# IDA自动分析器

在auto.hpp中，包含了关于IDA自动分析引擎相关的一些函数。

当加载一个新的二进制文件的时候，IDA的自动分析引擎便会开始工作。

IDA的自动分析器包含多个分析队列，每个队列有各自的优先级。当所有的分析队列都为空的时候IDA就会结束自动分析。



通过头文件中提供的接口，我们可以对自动分析器进行一些控制。此接口一般来说用的比较少。。。。。。

------

有的时候我们需要对自动分析引擎的状态进行判断，因为自动分析引擎的结果可能会影响到插件的使用。

这个时候可以使用**auto_is_ok**，该函数作用为判断分析队列是否全部为空。

```c++
 bool auto_is_ok(void);
```

示例代码如下:

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

------

还有一种情况是我们通过其它的API，比如使用add_func在某处汇编代码处生成了函数或者修改了某个导入函数的参数类型，这会使得自动分析队列添加新的分析任务。

这个时候我们想要等待自动分析引擎结束工作后，再执行插件的后续步骤，可以使用如下接口。

该函数的作用是阻塞等待，直到分析队列为空。

```c++
bool auto_wait(void);
```

由于此函数阻塞后会使得IDA处于假死状态，我们可以在调用该函数前加上一个提示窗，示例代码如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <auto.hpp>

bool idaapi run(size_t)
{
    //添加某个函数使得分析队列不为空
    add_func(0x405670);

    show_wait_box("Add some Function");
    auto_wait();
    hide_wait_box();
    
    return true;
}
```