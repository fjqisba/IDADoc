# Windows下开发环境配置:

1.建立一个空的Visual Studio项目，添加IDADemo.cpp文件

2.由于7.0版本以后的IDA插件必须都是64位的，因此编译平台选择x64

3.进行如下设置

配置属性->常规:更改"配置类型"为**动态库(.dll)**

C/C++ -> 常规:添加SDK的include路径到"附加包含目录"一栏，比如D:\idasdk74\include

C/C++ -> 预处理器:添加`__NT__`

C/C++ -> 代码生成:更改"安全检查"为**禁用安全检查(/GS-)**

链接器 -> 常规:添加SDK的lib库路径到"附加库目录"一栏，比如针对ida.exe所写的插件添加D:\idasdk74\lib\x64_win_vc_32，针对ida64.exe所写的插件添加D:\idasdk74\lib\x64_win_vc_64。

链接器 -> 附加依赖项: 添加`ida.lib`到"附加依赖项"一栏。



这样一个IDA插件开发环境就搭建好了。

### 插件模板介绍:

在IDADemo.cpp添加如下模板代码:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>

//以上是导入的SDK头文件

int idaapi init(void)
{
  //IDA在启动的时候会调用每个插件的init函数。
  //返回值有三种选项:
  //PLUGIN_SKIP适合那些不支持的插件，IDA将不会加载该插件
  //PLUGIN_OK适合那些执行一次性功能的插件
  //PLUGIN_KEEP适合那些需要一直保持功能的插件
  return PLUGIN_OK;
}

void idaapi term(void)
{
  //当结束插件时，一般您可以在此添加一点任务清理的代码。
  return;
}

bool idaapi run(size_t)
{
  //当按下热键时候,执行功能的入口函数
  warning("Hello, world!");
  return true;
}

static char comment[] = "It's a plugin to show Hello world!";

plugin_t PLUGIN =
{
  IDP_INTERFACE_VERSION,
  0,                    // 插件的一些属性,一般为0即可
  init,                 // initialize
  term,                 // terminate. this pointer may be NULL.
  run,                  // invoke plugin
  comment,              // 插件的说明,会显示在IDA下方的状态栏中
  "",                   // multiline help about the plugin
  "Hello, world",		// 插件在列表中显示的名称
  "Alt-F1"              // 插件想要注册的功能快捷键
};

```

编译运行该插件后，会显示出提示框Hello,world!

------

当IDA加载文件后，会生成一个idainfo信息，该信息存在在数据库文件中(即IDB文件)。

idainfo结构体定义在<ida.hpp>文件中，下面列出一部分值

```c++
struct idainfo
{
	char tag[3];		//固定为'IDA'
	char zero;		//没用
	ushort version;		//数据库版本
	char procname[16];	//当前处理器名称

	...
	ushort filetype;	// 被反汇编的文件类型,例如f_PE,f_ELF,参考filetype_t
	ea_t startIP;		// 程序开始运行时,[E]IP寄存器的值
	ea_t startSP;		// 程序开始运行时,[E]SP寄存器的值
	ea_t main;		// IDA解析出的主函数入口点的线性地址
	ea_t minEA;		// 程序的最小线性地址
	ea_t maxEA;		// 程序的最大线性地址
	...
};
```

该结构体以全局变量inf的形式定义在<ida.hpp>头文件中，我们可以直接使用。

比方说，我们想要编写的插件，只想处理Intel x86处理器类型下的PE和ELF两种格式的文件，编写以下代码:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>

int idaapi init(void)
{
	qstring ProcName = inf.procname;
	if (ProcName != "metapc" || (inf.filetype != f_ELF && inf.filetype != f_PE))
	{
		return PLUGIN_SKIP;
	}

	return PLUGIN_OK;
}
```

这样遇到不符合的文件，插件将不会载入。

------

