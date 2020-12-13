# CFunc

CFunc与反编译的函数有关。cfunc_t结构体用于存储反编译的结果。

------

#### get_line_item

函数原型如下:

```c++
bool get_line_item(const char *line, int x, bool is_ctree_line, ctree_item_t *phead, ctree_item_t *pitem, ctree_item_t *ptail);
```

该函数用于获取指定位置的ctree item。

参数一line为反编译文本中的某一行。

参数二x为反编译文本行中的横坐标。

参数三is_ctree_line表示是否在声明语句区域(如果不是，则假定为变量声明语句区域)。

参数四phead为返回的item的头部，用于附加块注释。可为空。

参数五pitem为返回的item。可为空。

参数六ptail为返回的item的尾部，用于附加缩进的注释。可为空。

如果没有获取到pitem，则函数返回值为false。



我们可以使用该函数给代码添加注释，示例代码如下:

```c++
#include <hexrays.hpp>

bool idaapi run(size_t)
{
	func_t* pfn = get_func(get_screen_ea());
	hexrays_failure_t hf;
	cfuncptr_t cfunc = decompile(pfn, &hf, DECOMP_WARNINGS);
	if (cfunc == NULL)
	{
		warning("error");
		return true;
	}

	const strvec_t& sv = cfunc->sv;
	
	//给函数的每行代码添加注释
	for (unsigned int n = 1; n < sv.size(); ++n)
	{
		ctree_item_t commentItem;
		cfunc->get_line_item(sv[n].line.c_str(), 0, true, NULL, NULL, &commentItem);

		qstring qComment;
		qComment.sprnt("%d", n);
		cfunc->set_user_cmt(commentItem.loc, qComment.c_str());
	}

	//添加注释后务必调用此函数保存注释
	cfunc->save_user_cmts();
	return true;
}
```

