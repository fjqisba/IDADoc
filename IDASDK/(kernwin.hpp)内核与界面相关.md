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

弹框提示信息相关的函数有:

```c++
//弹出一个错误窗口，然后退出IDA
void 	error(const char *format,...);

//弹出一个警告窗口
void 	warning(const char *format,...);

//弹出一个信息窗口
void 	info(const char *format,...);
```

------

弹出一个等待框，或者隐藏当前的等待框

```c++
void show_wait_box(const char *format, ...);
void hide_wait_box(void);
```

需要注意的是，如果多次调用show_wait_box函数，那么IDA会将当前文本存储到堆栈中然后显示出新的文本，因此show_wait_box和hide_wait_box函数必须成对出现，否则弹出的等待框会使得IDA界面无法进行任何交互。

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

------

创建一个选择窗口，可以使用choose函数，可以参考官方的choose.cpp示例，这里只介绍一些关键点

- 我们需要自定义一个结构体，然后继承chooser_t
- 重写类chooser_t的get_count函数，用来设置所显示列表的行数
- 重写类chooser_t的get_row函数，用来输出每一行内容
- (可选)重写类chooser_t的enter函数，用来设置双击选项执行的动作
- 申请chooser结构体变量必须使用new来创建，关闭选择窗口内存会自动释放

------

