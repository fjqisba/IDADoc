# 反汇编引擎

包含用来处理程序指令的反汇编相关函数。



一条指令的反汇编由以下三步组成:

1. 指令分析:ana.cpp
2. 模拟分析:emu.cpp
3. 转换为反汇编文本:out.cpp

------

判断指定地址的字节是否可以被解码为一条有效指令

```c++
bool can_decode (ea_t ea)
```

参数一ea表示指定的线性地址。

函数返回true表示可以被解码为有效指令。

------

对一条指令进行反汇编，使用以下函数

```c++
int decode_insn (insn_t *out, ea_t ea)
```

参数一out为返回的指令信息结果

参数二ea为要进行反汇编的指令地址。

返回值为指令的长度，如果为0则表示反汇编失败。

------

IDA还提供了一个强大的逆反汇编功能，可以用以下函数对指定地址的上一条指令进行反汇编

```c++
ea_t decode_prev_insn (insn_t *out, ea_t ea)
```

参数一out为返回的指令信息结果

参数二ea表示要从哪个地址开始反汇编上一条指令

返回值为上一条指令的地址，如果返回**BADADDR**表示反汇编失败。

------

`insn_t`代表着指令的反汇编结果，该结构体部分成员如下

```c++
class insn_t
{
    ...
    ea_t ea;			//该指令的线性地址
    uint16 itype;		//内部指令类型,由IDP来定义
    uint16 size;		//指令长度
    ...
    op_t ops[UA_MAXOP];		//操作数数组
}
```

我们得到insn_t后，一般先通过判断itype来确定指令的类型，这个itype是个枚举类型，不同的处理器对应的枚举类型都不同，需要自行去allins.hpp中寻找。

得到指令的类型后我们就可以通过解析ops数组来进一步得到详细的信息了。

ops数组大小固定为8，但大部分指令的有效op并没有这么多。

现假设在X86平台下，我们需要打印出某个函数里面call Register、call [MemAddr]这两种形式的全部指令。

示例代码如下:

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <ua.hpp>
#include <allins.hpp>

qstring RegToName(uint16 reg)
{
	qstring ret;
	switch (reg)
	{
	case 0x0:
		ret = "eax";
		break;
	case 0x1:
		ret = "ecx";
		break;
	case 0x2:
		ret = "edx";
		break;
	case 0x3:
		ret = "ebx";
		break;
	case 0x4:
		ret = "esp";
		break;
	case 0x5:
		ret = "ebp";
		break;
	case 0x6:
		ret = "esi";
		break;
	case 0x7:
		ret = "edi";
		break;
	default:
		break;
	}

	return ret;
}

bool idaapi run(size_t)
{
	//假设函数之间的指令是连续的

	//某个函数的起始地址
	ea_t FuncStart = 0x00401180;
	int iLen = 0;
	do
	{
		insn_t ins;
		iLen = decode_insn(&ins, FuncStart);
		if (ins.itype == NN_callni)
		{
			//call eax这类的指令
			if (ins.ops[0].type == o_reg)
			{
				qstring Reg = RegToName(ins.ops[0].reg);
				msg("[Instruction]:%a---call %s\n", FuncStart, Reg.c_str());
				FuncStart += iLen;
				continue;
			}

			//call [MemAddr]
			if (ins.ops[0].type == o_mem)
			{
				msg("[Instruction]:%a---call [%a]\n", FuncStart, ins.ops[0].addr);
				FuncStart += iLen;
				continue;
			}
		}
		FuncStart += iLen;
	} while (iLen);
	return true;
}
```

------

