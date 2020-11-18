# 内核与界面相关

<kernwin.hpp>中定义了IDA的内核与界面之间的接口。



输出窗口相关的函数有:

```c++
//清空输出窗口内容
void	msg_clear();

//将输出窗口内容保存到文件中
bool	msg_save(qstring &path);

//打印格式化文本至输出窗口,类似于printf
int 	msg(const char *format,...);
```

------

弹框相关的函数有:

```c++
//弹出一个错误窗口，然后退出IDA
void 	error(const char *format,...);

//弹出一个警告窗口
void 	warning(const char *format,...);

//弹出一个信息窗口
void 	info(const char *format,...);
```

------

跳转函数如下，其余的可选参数基本没什么用。。。。

```c++
#define 	UIJMP_ACTIVATE   0x0001
#define 	UIJMP_DONTPUSH   0x0002
#define 	UIJMP_IDAVIEW    0x0004
bool jumpto(ea_t ea, int opnum=-1, int uijmp_flags=UIJMP_ACTIVATE)
```

------

获取当前屏幕光标处的地址，这个函数也算是一个交互函数吧，比如可以用来获取用户指向的代码/数据位置。

```c++
ea_t 	get_screen_ea(void);
```

