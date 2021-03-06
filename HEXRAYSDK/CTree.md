# CTree

在理解CTree的核心之前，我们还需要认识以下类:

**cfunc_t**:cfunc_t中包含着和反编译代码相关的一些基础信息，cfunc_t中存在一个body成员，是一个cinsn_t类型，代表整个反编译函数的主体。

```c++
struct cfunc_t
{
  ea_t entry_ea;             ///< function entry address
  cinsn_t body;              ///< 函数的主体, 一定是个cblock类型
  //...
}
```

**cinsn_t**:cinsn_t代表着用来组成CTree的每条语句，结构体内的节点主要和流程控制有关。

```c++
struct cinsn_t : public citem_t
{
  union
  {
    cblock_t *cblock;   ///< details of block-statement
    cexpr_t *cexpr;     ///< details of expression-statement
    cif_t *cif;         ///< details of if-statement
    cfor_t *cfor;       ///< details of for-statement
    cwhile_t *cwhile;   ///< details of while-statement
    cdo_t *cdo;         ///< details of do-statement
    cswitch_t *cswitch; ///< details of switch-statement
    creturn_t *creturn; ///< details of return-statement
    cgoto_t *cgoto;     ///< details of goto-statement
    casm_t *casm;       ///< details of asm-statement
  };
}
```

**cexpr_t**:cexpr_t代表着用来组成每条语句的C语言表达式，可能也是最常用的一个类，里面存储着每条表达式的关键信息。

```c++
struct cexpr_t : public citem_t
{
  union
  {
    cnumber_t *n;     ///< used for \ref cot_num
    fnumber_t *fpc;   ///< used for \ref cot_fnum
    struct
    {
      union
      {
        var_ref_t v;  ///< used for \ref cot_var
        ea_t obj_ea;  ///< used for \ref cot_obj
      };
      int refwidth;   ///< how many bytes are accessed? (-1: none)
    };
    struct
    {
      cexpr_t *x;     ///< the first operand of the expression
      union
      {
        cexpr_t *y;   ///< the second operand of the expression
        carglist_t *a;///< argument list (used for \ref cot_call)
        uint32 m;     ///< member offset (used for \ref cot_memptr, \ref cot_memref)
                      ///< for unions, the member number
      };
      union
      {
        cexpr_t *z;   ///< the third operand of the expression
        int ptrsize;  ///< memory access size (used for \ref cot_ptr, \ref cot_memptr)
      };
    };
    cinsn_t *insn;    ///< an embedded statement, they are prohibited
                      ///< at the final maturity stage (\ref CMAT_FINAL)
    char *helper;     ///< helper name (used for \ref cot_helper)
    char *string;     ///< string constant (used for \ref cot_str)
  };
  tinfo_t type;       ///< expression type. must be carefully maintained
}
```

**citem_t**:citem_t是cinsn_t和cexpr_t的基类，用来存储一些通用的信息，例如地址、标签。



为了更进一步地了解CTree，以下代码展示了如何去遍历函数的CTree节点，并且实现了一个C语句表达式的模板化引擎。

```c++
#include <hexrays.hpp>

qstring GetExprString(cexpr_t* pItem)
{
	qstring ret;

	switch (pItem->op)
	{
	case cot_add:
		return GetExprString(pItem->x) + "+" + GetExprString(pItem->y);
	case cot_asg:
		return GetExprString(pItem->x) + "=" + GetExprString(pItem->y);
	case cot_asgadd:
		return GetExprString(pItem->x) + "+=" + GetExprString(pItem->y);
	case cot_band:
		return  GetExprString(pItem->x) + "&" + GetExprString(pItem->y);
	case cot_call:
		ret = "call(";
		for (unsigned int n = 0; n < pItem->a->size(); ++n)
		{
			ret += GetExprString(&pItem->a->at(n)) + ",";
		}
		if (pItem->a->size())
		{
			ret.remove_last();
		}
		ret += ")";
		return ret;
	case cot_cast:
		return qstring("(cast)") + GetExprString(pItem->x);
	case cot_eq:
		return GetExprString(pItem->x) + "==" + GetExprString(pItem->y);
	case cot_idx:
		return GetExprString(pItem->x) + qstring("[") + GetExprString(pItem->y) + qstring("]");
	case cot_mul:
		return GetExprString(pItem->x) + "*" + GetExprString(pItem->y);
	case cot_num:
		ret = "num";
		break;
	case cot_obj:
		ret = "obj";
		break;
	case cot_ref:
		return qstring("&") + GetExprString(pItem->x);
	case cot_var:
		ret = "var";
		break;
	case cot_preinc:
		return qstring("++") + GetExprString(pItem->x);
		break;
	case cot_ptr:
		return qstring("*") + GetExprString(pItem->x);
		break;
	case cot_sub:
		return GetExprString(pItem->x) + "-" + GetExprString(pItem->y);
	case cot_tern:
		return GetExprString(pItem->x) + "?" + GetExprString(pItem->y) + ":" + GetExprString(pItem->z);
	case cot_ult:
		return GetExprString(pItem->x) + "<" + GetExprString(pItem->y);
	case cot_xor:
		return  GetExprString(pItem->x) + "^" + GetExprString(pItem->y);
	default:
		//遇到没解析过的类型自行补充就行了
		msg("UnHandled Item Type...\n");
		break;
	}

	return ret;
}

//--------------------------------------------------------------------------
bool idaapi run(size_t)
{
	func_t* pfn = get_func(get_screen_ea());
	hexrays_failure_t hf;
	cfuncptr_t cfunc = decompile(pfn, &hf, DECOMP_WARNINGS);
	if (cfunc == NULL)
	{
		warning("error");
		return true;
	}

	ctree_items_t& vec_TreeItem = cfunc->treeitems;
	for (unsigned int n = 0; n < vec_TreeItem.size(); ++n)
	{
		//打印出所有的表达式吧
		if (vec_TreeItem[n]->op == cit_expr)
		{
			qstring ExprPatterm = GetExprString(((cexpr_t*)vec_TreeItem[n])->x);
			msg("%s\n", ExprPatterm.c_str());
		}
	}

    return true;
}
```
