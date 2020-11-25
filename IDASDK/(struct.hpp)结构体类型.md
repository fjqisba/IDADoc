# 结构体类型

IDA内部维护着一个结构体列表，IDA中的Structures窗口可以看到结构体列表的信息。

------

创建结构体

```c
tid_t add_struc(uval_t idx, const char *name, bool is_union=false);
```

第一个参数，idx为结构体的索引，代表所插入的结构体在IDA存储的列表中的顺序，如果idx为-1，表示添加结构体到列表末尾。

第二个参数，name为结构体的名称，不能为不合法名称，也不可以与数据库已有的名称重复。

返回值tid_t实际上就是个整数，用于标识唯一的结构体类型。如果返回-1，代表创建结构体失败。

------

通过标识符tid_t索引得到对应的结构体类型信息struc_t。

```c
struc_t* get_struc(tid_t id);
```

通过结构体的名称来索引得到对应的结构体ID，或者通过结构体ID索引得到结构体的名称。

```c
tid_t get_struc_id(const char *name);
qstring get_struc_name(tid_t id);
```

------

扩充或者收缩一个结构体

```c
bool expand_struc(struc_t *sptr, ea_t offset, adiff_t delta, bool recalc=true);
```

##### 参数:

sptr表示需要执行操作的结构体。

offset表示结构体的偏移。

delta表示要扩充的字节大小，如果为负数则表示要移除的字节大小。

recalc表示是否重新计算结构体类型被使用的地方。

##### 返回值:

返回true表示函数执行成功。



需要注意的是，一个空的结构体是无法执行expand_struc函数的，可以参考以下代码初始化一个结构体

```c
//name为结构体的名称,size为结构体的大小
tid_t CreateStructure(const char* name, int size)
{
    //已存在相同名称的结构体
    tid_t structId = get_struc_id(name);
    if (structId != BADADDR)
    {
        return structId;
    }

    structId = add_struc(-1, name);
    if (structId == BADADDR)
    {
        return structId;
    }

    struc_t* pStruct = get_struc(structId);
    char fieldName[64] = { 0 };
    for (int n = 0; n < size; ++n)
    {
        qsnprintf(fieldName, sizeof(fieldName), "field_%a", n);
        add_struc_member(pStruct, fieldName, n, 0, NULL, 1);
    }

    return structId;
}
```





