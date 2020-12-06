# HexRays SDK简介
IDA反编译器Hex-Rays是作为IDA的插件而存在的，它可以将IDA反汇编代码转换成类C语言伪代码。

同时官方还提供了一组HexRays SDK，供用户使用。



SDK使用方法很简单，只需要将hexrays.hpp文件包含即可。



在HexRays反编译器中二进制代码有两种存在的形式，一种是microcode，一种是ctree。

**microcode**:微码，由一条一条的处理器指令组成，由反编译器来进行优化并转换。

**ctree**:Ctree由经过优化过后的微码生成，其内部其实是一个用C语句和表达式构建的类抽象语法树。Ctree可以转换为伪C代码。

------

官方示例代码如下:

```C++
#include <hexrays.hpp>

// Hex-Rays API pointer
hexdsp_t *hexdsp = NULL;

static bool inited = false;
//--------------------------------------------------------------------------
int idaapi init(void)
{
  //初始化hexrays插件
  if ( !init_hexrays_plugin() )
  {
       return PLUGIN_SKIP; // no decompiler
  }
  inited = true;
  return PLUGIN_KEEP;
}

//--------------------------------------------------------------------------
void idaapi term(void)
{
  //释放hexrays插件
  if ( inited )
  {
     term_hexrays_plugin(); 
  }
}

//--------------------------------------------------------------------------
bool idaapi run(size_t)
{
  func_t *pfn = get_func(get_screen_ea());
  if ( pfn == NULL )
  {
    warning("Please position the cursor within a function");
    return true;
  }
  hexrays_failure_t hf;
  cfuncptr_t cfunc = decompile(pfn, &hf, DECOMP_WARNINGS);
  if ( cfunc == NULL )
  {
    warning("#error \"%a: %s", hf.errea, hf.desc().c_str());
    return true;
  }
  msg("%a: successfully decompiled\n", pfn->start_ea);

  const strvec_t &sv = cfunc->get_pseudocode();
  for ( int i=0; i < sv.size(); i++ )
  {
    qstring buf;
    tag_remove(&buf, sv[i].line);
    msg("%s\n", buf.c_str());
  }
  return true;
}

//--------------------------------------------------------------------------
static char comment[] = "Sample1 plugin for Hex-Rays decompiler";

plugin_t PLUGIN =
{
  IDP_INTERFACE_VERSION,
  0,                    // plugin flags
  init,                 // initialize
  term,                 // terminate. this pointer may be NULL.
  run,                  // invoke plugin
  comment,              // long comment about the plugin
                        // it could appear in the status line
                        // or as a hint
  "",                   // multiline help about the plugin
  "Decompile & Print",  // the preferred short name of the plugin
  ""                    // the preferred hotkey to run the plugin
};
```

上述代码的作用就是打印出当前窗口所在函数的伪代码文本。

除了执行上面的代码稍微体验一下HexRays的强大之外，我们还需要知道以下几点：

1.cfunc_t可以理解为操作反编译器API功能的入口，很多功能都要依赖这个类来实现

2.**decompile**函数几乎是我们主动获取cfunc_t的唯一接口。