# IDC脚本简介

IDC脚本是由IDA官方提供并维护的功能，语法和C语言比较相似。

------

在IDA中，我们可以点击菜单File -> Script File来执行一个IDC脚本。

点击菜单File -> Script Command可以开启一个脚本执行窗口，我们可以在这里编写脚本测试运行。

例如编写下列代码后点击Run按钮。

```c
msg("Hello World\n");
```

IDA的Output窗口便会输出文本结果。

因为是脚本型的语言，如果我们没有定义函数，脚本则默认从头开始执行。

如果我们想要定义函数，那么就得使用下面这种方式:

```c
static GetText()
{
   auto Text="Hello World\n";
   return Text;
}

static main()
{
    msg(GetText());
}
```

事实上，函数的声明都遵守以下格式:

**static func(arg1,arg2,arg3)**

而且由于IDC脚本的变量都是`auto`通用类型，一个变量可以保存任何类型的数据，因此函数没必要定义返回类型，所有函数默认都有返回值。

------

了解了以上常识，就能解决大部分编写脚本的需求了，下面是一些特殊的点:

- 全局变量使用`extern`关键字命名，且不可进行初始化赋值
- 不支持C语言中的switch case语句，这一点和Python很像。