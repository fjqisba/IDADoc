# 名称相关

name.hpp中包含一些用于处理名称的函数。



设置或者删除指定地址处的项目名称。

```c++
bool set_name (ea_t ea, const char *name, int flags=0)
```

参数一为线性地址。

参数二为设置的新名称，如果为"",则表示删除名称。

返回真表示函数执行成功。

------

获取指定地址的名称

```c++
ssize_t get_ea_name (qstring *out, ea_t ea, int gtn_flags=0, getname_info_t *gtni=NULL)
```

参数一用于接收返回的名称。

参数二为指定的地址。

------

对名称进行解码，此函数一般用来解码C++的符号。

如不清楚什么是解码，可以参考[Name mangling - Wikipedia](https://en.wikipedia.org/wiki/Name_mangling)

```c++
qstring demangle_name(
        const char *name,
        uint32 disable_mask,
        demreq_type_t demreq=DQT_FULL)
int32  demangle_name(
        qstring *out,
        const char *name,
        uint32 disable_mask,
        demreq_type_t demreq=DQT_FULL);
```

第一个参数name是解码的文本

第二个参数disable_mask是解码的一些选项，和IDA主程序Options->Demangled names里面的选项差不多。

第三个参数demreq是想要得到的解码结果。

------

