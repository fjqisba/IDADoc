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

------

弹出选择窗口，一般使用choose函数，这里我们可以参考官方的choose.cpp示例

```c++
#include <ida.hpp>
#include <idp.hpp>
#include <loader.hpp>
#include <bytes.hpp>
#include <kernwin.hpp>

//我们需要自定义一个结构体,并且继承chooser_t
struct calls_chooser_t : public chooser_t
{
protected:
	static const int widths_[];
	static const char* const header_[];

public:
	//存储的数据
	qvector<ea_t> list;

	//对象必须使用new来构造
	calls_chooser_t(const char* title, bool ok, func_item_iterator_t* fii);

	//重写该函数,用于设置显示列表的行数
	virtual size_t idaapi get_count() const
	{
		return list.size();
	}

	//重写该函数,用于输出每一行内容
	virtual void idaapi get_row(
		qstrvec_t* cols_,
		int* icon_,
		chooser_item_attrs_t* attrs,
		size_t n) const;
	{
		// assert: n < list.size()
		ea_t ea = list[n];

		// generate the line
		qstrvec_t& cols = *cols_;
		cols[0].sprnt("%08a", ea);
		generate_disasm_line(&cols[1], ea, GENDSM_REMOVE_TAGS);
	}
	//重写该函数,用于设置双击选项执行的动作
	virtual cbret_t idaapi enter(size_t n)
	{
		if (n < list.size())
			jumpto(list[n]);
		return cbret_t(); // nothing changed
	}

protected:
	void build_list(bool ok, func_item_iterator_t* fii)
	{
		insn_t insn;
		while (ok)
		{
			ea_t ea = fii->current();
			if (decode_insn(&insn, ea) > 0 && is_call_insn(insn)) // a call instruction is found
				list.push_back(ea);
			ok = fii->next_code();
		}
	}
};

//每一列的宽度
const int calls_chooser_t::widths_[] =
{
  CHCOL_HEX | 8,  // Address
  32,             // Instruction
};
//每一列的标题
const char* const calls_chooser_t::header_[] =
{
  "Address",      // 0
  "Instruction",  // 1
};

inline calls_chooser_t::calls_chooser_t(
	const char* title_,
	bool ok,
	func_item_iterator_t* fii)
	: chooser_t(0, qnumber(widths_), widths_, header_, title_),
	list()
{
	CASSERT(qnumber(widths_) == qnumber(header_));

	// build the list of calls
	build_list(ok, fii);
}


bool idaapi run(size_t)
{
	qstring title;
	// Let's display the functions called from the current one
	// or from the selected area

	// First we determine the working area

	func_item_iterator_t fii;
	bool ok;
	ea_t ea1, ea2;
	if (read_range_selection(NULL, &ea1, &ea2)) // the selection is present?
	{
		callui(ui_unmarksel);                       // unmark selection
		title.sprnt("Functions called from %08a..%08a", ea1, ea2);
		ok = fii.set_range(ea1, ea2);
	}
	else                                          // nothing is selected
	{
		func_t* pfn = get_func(get_screen_ea());    // try the current function
		if (pfn == NULL)
		{
			warning("Please position the cursor on a function or select an area");
			return true;
		}
		ok = fii.set(pfn);
		get_func_name(&title, pfn->start_ea);
		title.insert("Functions called from ");
	}

	//需要申请类,关闭窗口会自动释放
	calls_chooser_t* ch = new calls_chooser_t(title.c_str(), ok, &fii);
 
	//调用choose函数显示出窗口
	ch->choose(); // default cursor position is 0 (first row)
	return true;
}
```

