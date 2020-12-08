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

通过结构体的名称来索引到对应的结构体标识符:

```c++
tid_t get_struc_id(const char *name);
```

根据结构体标识符来索引到对应的结构体名称:

```c++
qstring get_struc_name(tid_t id);
```

------

设置结构体的注释和获取结构体的注释可参考如下函数

```c++
bool set_struc_cmt(tid_t id, const char *cmt, bool repeatable);
ssize_t get_struc_cmt(qstring *buf, tid_t id, bool repeatable);
```

------

通过标识符tid_t索引得到对应的结构体类型信息struc_t:

```c
struc_t* get_struc(tid_t id);
```

------

获取一个结构体的大小

```c++
asize_t 	get_struc_size (const struc_t *sptr);
asize_t 	get_struc_size (tid_t id);
```

------

给结构体添加成员

```c++
struc_error_t  add_struc_member(
        struc_t *sptr,
        const char *fieldname,
        ea_t offset,
        flags_t flag,
        const opinfo_t *mt,
        asize_t nbytes);
```

第一个参数sptr为要添加成员的结构体

第二个参数fieldname为要添加的成员名称

第三个参数offset为要添加的成员在结构体中的偏移

第四个参数flag为结构体的类型标记，可在bytes.hpp中找到如下定义

```c++
flags_t idaapi byte_flag(void)     { return FF_DATA|FF_BYTE; }      ///< Get a flags_t representing a byte
flags_t idaapi word_flag(void)     { return FF_DATA|FF_WORD; }      ///< Get a flags_t representing a word
flags_t idaapi dword_flag(void)    { return FF_DATA|FF_DWORD; }     ///< Get a flags_t representing a double word
flags_t idaapi qword_flag(void)    { return FF_DATA|FF_QWORD; }     ///< Get a flags_t representing a quad word
flags_t idaapi oword_flag(void)    { return FF_DATA|FF_OWORD; }     ///< Get a flags_t representing a octaword
flags_t idaapi yword_flag(void)    { return FF_DATA|FF_YWORD; }     ///< Get a flags_t representing a ymm word
flags_t idaapi zword_flag(void)    { return FF_DATA|FF_ZWORD; }     ///< Get a flags_t representing a zmm word
flags_t idaapi tbyte_flag(void)    { return FF_DATA|FF_TBYTE; }     ///< Get a flags_t representing a tbyte
flags_t idaapi strlit_flag(void)   { return FF_DATA|FF_STRLIT; }    ///< Get a flags_t representing a string literal
flags_t idaapi stru_flag(void)     { return FF_DATA|FF_STRUCT; }    ///< Get a flags_t representing a struct
flags_t idaapi cust_flag(void)     { return FF_DATA|FF_CUSTOM; }    ///< Get a flags_t representing custom type data
flags_t idaapi align_flag(void)    { return FF_DATA|FF_ALIGN; }     ///< Get a flags_t representing an alignment directive
flags_t idaapi float_flag(void)    { return FF_DATA|FF_FLOAT; }     ///< Get a flags_t representing a float
flags_t idaapi double_flag(void)   { return FF_DATA|FF_DOUBLE; }    ///< Get a flags_t representing a double
flags_t idaapi packreal_flag(void) { return FF_DATA|FF_PACKREAL; }  ///< Get a flags_t representing a packed decimal real
```

第五个参数mt为要添加的成员类型的额外信息，只有当成员类型为结构体、偏移、枚举、字符串、结构体偏移以上一种时有效。

第六个参数nbytes为要添加的成员的大小，大小必须和flag对应，例如当参数flag填`dword_flag()`时，nbytes应该填4。

当填0的时候，该成员会表示成一个`变量结构体`（即大小不确定）。

------

设置结构体某个成员的名称

```c++
bool set_member_name(struc_t *sptr, ea_t offset,const char *name);
```

参数一sptr为要目标成员的结构体

参数二offset为目标成员所处在结构体的偏移大小

参数三name为目标成员要设置的名称

------

设置变量的类型，该函数可用于设置成员为一些基础类型(byte,word,dword...)，参数和add_struc_member差不多，故省略介绍。

```c++
bool set_member_type(struc_t *sptr, ea_t offset, flags_t flag,const opinfo_t *mt, asize_t nbytes);
```



如果要将结构体成员设置为类型库中的其它类型，可使用以下函数

```c++
smt_code_t set_member_tinfo(
        struc_t *sptr,
        member_t *mptr,
        uval_t memoff,
        const tinfo_t &tif,
        int flags);
```

参数一sptr为成员所在的结构体

参数二mptr为目标成员

参数三memoff成员内部的偏移

参数四tif为要设置的成员类型

参数五flags为设置的一些参数，有如下选择

```c++
#define SET_MEMTI_MAY_DESTROY 0x0001 ///< may destroy other members
#define SET_MEMTI_COMPATIBLE  0x0002 ///< new type must be compatible with the old
#define SET_MEMTI_FUNCARG     0x0004 ///< mptr is function argument (can not create arrays)
#define SET_MEMTI_BYTIL       0x0008 ///< new type was created by the type subsystem
#define SET_MEMTI_USERTI      0x0010 ///< user-specified type
```



例如我们想要创建一个结构体，结构体第一个成员是自身的指针，代码如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <struct.hpp>
#include <typeinf.hpp>

til_t* idati;
bool idaapi run(size_t)
{
	idati = (til_t*)get_idati();

	tid_t idStruct = add_struc(-1, "TestStruct");
	struc_t* pStruct = get_struc(idStruct);

	add_struc_member(pStruct, "member_0", 0, dword_flag(), NULL, 0x4);

	tinfo_t type;
	type.create_typedef(idati, "TestStruct");
	type.create_ptr(type);
	set_member_tinfo(pStruct, get_member(pStruct, 0), 0, type, SET_MEMTI_MAY_DESTROY);
	return true;
}
```

------

如果要遍历一个结构体的成员，可以参考如下代码:

```c++
bool idaapi run(size_t)
{
	//要遍历的某个结构体的名称
	struc_t* pStruct = get_struc(get_struc_id("GUID"));
	if (!pStruct)
	{
		return true;
	}
	for (unsigned int n = 0; n < pStruct->memqty; ++n)
	{
		member_t pMember = pStruct->members[n];
		qstring qMemberName = get_member_name(pMember.id);
		msg("[Member]:%s\n", qMemberName.c_str());
	}

	return true;
}
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
