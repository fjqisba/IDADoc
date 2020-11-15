# 枚举类型

enum.hpp包含了IDA中枚举信息相关的接口。

枚举类型或者位域(bit fields)表示为enum_t，实际上等价于tid_t，代表着一种数据类型的标识符。



IDA内部推测维护着一个枚举类型列表

对枚举类型进行遍历，示例代码如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <enum.hpp>

struct MyVisitor:public enum_member_visitor_t
{
	//函数返回非0值,代表终止遍历枚举类型
	virtual int idaapi visit_enum_member(const_t cid, uval_t value)
	{
		qstring MemberName;
		get_enum_member_name(&MemberName, cid);
		msg("Member:%s----%d\n", MemberName.c_str(), value);
		return 0;
	}
};

bool idaapi run(size_t)
{
	//获取enum_t个数
	size_t enumCount = get_enum_qty();
	for (unsigned int idx = 0; idx < enumCount; ++idx)
	{
		enum_t enumId = getn_enum(idx);

		qstring enumName = get_enum_name(enumId);

		msg("%s\n", enumName.c_str());

		MyVisitor EnumVisitor;
		for_all_enum_members(enumId, EnumVisitor);
		msg("----\n");
 	}

	return true;
}
```



