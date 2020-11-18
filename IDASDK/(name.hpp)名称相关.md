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

