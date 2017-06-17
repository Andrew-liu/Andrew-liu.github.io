title: Python源码剖析读书笔记(二)
date: 2015-01-16 14:06:22
tags: [Python, 读书笔记]
---

> 第二部分看完了, 但是字节码部分还是有很多不懂的地方, 这部分等以后有长进了在回来读一遍...


#Python虚拟机
---

#1. Python的编译结果
---

> Code对象和pyc文件, Python执行原理`虚拟机, 字节码`

当输入命令`python my_program.py`时:

1. python解释器被激活执行python程序
2. 编译.py文件(生成字节码)
3. 将编译结果交给虚拟机

##1.1. PyCodeObject对象

> 编译结果存在内存中的`PyCodeObject`对象中, python运行结束后, 编译结果又被保存到pyc文件中, 下一次运行相同程序,python会根据pyc文件中的编译结果直接建立内存中`PyCodeObject`对象, 不再对源文件编译


<!--more-->

```py
#code.h
/* Bytecode object 字节码对象 */
typedef struct {
    PyObject_HEAD
    int co_argcount;        /* #arguments, except *args位置参数的个数 */
    int co_nlocals;     /* #local variables 局部变量个数 */
    int co_stacksize;       /* #entries needed for evaluation stack 执行code block需要的栈空间*/
    int co_flags;       /* CO_..., see below */
    PyObject *co_code;      /* instruction opcodes 字节码指令序列*/
    PyObject *co_consts;    /* list (constants used) 所有常量*/
    PyObject *co_names;     /* list of strings (names used) 所有符号*/
    PyObject *co_varnames;  /* tuple of strings (local variable names) 局部变量名集合*/
    PyObject *co_freevars;  /* tuple of strings (free variable names) 闭包需要的东西*/
    PyObject *co_cellvars;      /* tuple of strings (cell variable names) 内部嵌套函数引用的局部变量名集合*/
    /* The rest doesn't count for hash/cmp */
    PyObject *co_filename;  /* string (where it was loaded from) .py的完整路径*/
    PyObject *co_name;      /* string (name, for reference) 函数名或雷鸣*/
    int co_firstlineno;     /* first source line number .py文件起始行*/
    PyObject *co_lnotab;    /* string (encoding addr<->lineno mapping) See
                   Objects/lnotab_notes.txt for details.字节码与.py源码行号对应关系 */
    void *co_zombieframe;     /* for optimization only (see frameobject.c) */
    PyObject *co_weakreflist;   /* to support weakrefs to code objects */
} PyCodeObject;
```


> 代码中一个Code Block(对应一个名字空间)创建一个`PyCodeObject`对象(包含Code
Block编译后的byte code)

名字空间: 符号的上下文(一个变量名对应的变量值是什么需要通过名字空间决定)

一般对于有import语句的程序, 例如import abc, 运行时会先到设定好的路径找abc.pyc或abc.dll, 如果没有会先将abc.py编译成响应的`PyCodeObject`对象, 然后创建abc.pyc并将中间结果写入文件, 接下来python才对abc.pyc进行import操作(`PyCodeObject`对象复制到内存)

##1.2. pyc文件的生成

```py
#import.c
#define MAGIC (62211 | ((long)'\r'<<16) | ((long)'\n'<<24))
static void
write_compiled_module(PyCodeObject *co, char *cpathname, struct stat *srcstat, time_t mtime)
{
    FILE *fp;  #排他性的打开文件
#ifdef MS_WINDOWS   /* since Windows uses different permissions  */
    mode_t mode = srcstat->st_mode & ~S_IEXEC;
    /* Issue #6074: We ensure user write access, so we can delete it later
     * when the source file changes. (On POSIX, this only requires write
     * access to the directory, on Windows, we need write access to the file
     * as well)
     */
    mode |= _S_IWRITE;
#else
    mode_t mode = srcstat->st_mode & ~S_IXUSR & ~S_IXGRP & ~S_IXOTH;
#endif

    fp = open_exclusive(cpathname, mode);
    if (fp == NULL) {
        if (Py_VerboseFlag)
            PySys_WriteStderr(
                "# can't create %s\n", cpathname);
        return;
    }
    #写入python的magic number(保证兼容性)
    PyMarshal_WriteLongToFile(pyc_magic, fp, Py_MARSHAL_VERSION);
    /* First write a 0 for mtime */
    PyMarshal_WriteLongToFile(0L, fp, Py_MARSHAL_VERSION);
    #写入PyCodeObject对象
    PyMarshal_WriteObjectToFile((PyObject *)co, fp, Py_MARSHAL_VERSION);
    if (fflush(fp) != 0 || ferror(fp)) {
        if (Py_VerboseFlag)
            PySys_WriteStderr("# can't write %s\n", cpathname);
        /* Don't keep partial file */
        fclose(fp);
        (void) unlink(cpathname);
        return;
    }
    /* Now write the true mtime (as a 32-bit field)写入时间 */
    fseek(fp, 4L, 0);
    assert(mtime <= 0xFFFFFFFF);
    PyMarshal_WriteLongToFile((long)mtime, fp, Py_MARSHAL_VERSION);
    fflush(fp);
    fclose(fp);
    if (Py_VerboseFlag)
        PySys_WriteStderr("# wrote %s\n", cpathname);
}

#marshal.c
void
PyMarshal_WriteObjectToFile(PyObject *x, FILE *fp, int version)
{
    WFILE wf;
    wf.fp = fp;
    wf.error = WFERR_OK;
    wf.depth = 0;
    wf.strings = (version > 0) ? PyDict_New() : NULL;
    wf.version = version;
    w_object(x, &wf); #借用这个完成PyCodeObject对象写入pyc文件的操作
    Py_XDECREF(wf.strings);
}
```

将PyCodeObject对象写入pyc文件会调用write_compiled_module函数, 这个函数又会调用PyMarshal_WriteLongToFile函数, PyMarshal_WriteLongToFile函数会调用w_object函数, w_object函数会对不同对象执行不同的写入文件操作(其中会写入对象类型), 其中*WFILE中strings在向pyc写入时会指向PyDictObject对象, 从pyc读出时调用PyMarshal_ReadLongToFile函数, strings会指向PyListObject


字符串对象写入pyc有三种情况:

1. 普通字符串, 写入类型, 长度和字符串
2. intern字符串写入先进行查找, 查找失败进行首次写入, 将(字符串, 序号)添加到strings中, 将类型和字符串本身写入pyc
3. 查找成功进行非首次写入,仅仅将类型和查到的序号写入pyc中

字符串从pyc读操作:

1. 读好TYPE_INTERN后, 字符串进行intern操作, 并将intern操作结果添加到strings指向的PyListObject中
2. 当读到TYPE_STRINGREF, 后根据跟随的序号(写入pyc使生成的)访问strings, 从来获得进行了intern操作的字符串对象


> pyc中的PyCodeObject对象是以一种嵌套的关系联系在一起, 意味着pyc文件中的二进制数据有结构, 则可以进行可视化


#2. Python虚拟机架构
---

> python虚拟机模拟操作系统运行可执行文件的过程, 其中`PyCodeObject`没有包含程序`执行环境`

##2.1. PyFrameObject

```py
#frameobject.h
typedef struct _frame {
    PyObject_VAR_HEAD   #变长对象, 不同PyCodeObject, 栈空间大小不同
    struct _frame *f_back;  /* previous frame, or NULL 执行环境链上的前一个frame*/
    PyCodeObject *f_code;   /* code segment PyCodeObject对象*/
    PyObject *f_builtins;   /* builtin symbol table (PyDictObject) buildin名字空间*/
    PyObject *f_globals;    /* global symbol table (PyDictObject) global名字空间*/
    PyObject *f_locals;     /* local symbol table (any mapping) local名字空间*/
    PyObject **f_valuestack;    /* points after the last local 运行时栈的栈底位置*/
    /* Next free slot in f_valuestack.  Frame creation sets to f_valuestack.
       Frame evaluation usually NULLs it, but a frame that yields sets it
       to the current stack top. */
    PyObject **f_stacktop;  #运行时栈的栈底位置
    PyObject *f_trace;      /* Trace function */

    /* If an exception is raised in this frame, the next three are used to
     * record the exception info (if any) originally in the thread state.  See
     * comments before set_exc_info() -- it's not obvious.
     * Invariant:  if _type is NULL, then so are _value and _traceback.
     * Desired invariant:  all three are NULL, or all three are non-NULL.  That
     * one isn't currently true, but "should be".
     */
    PyObject *f_exc_type, *f_exc_value, *f_exc_traceback;

    PyThreadState *f_tstate;
    int f_lasti;        /* Last instruction if called 上一条字节码指令在f_code中的偏移位置*/
    /* Call PyFrame_GetLineNumber() instead of reading this field
       directly.  As of 2.3 f_lineno is only valid when tracing is
       active (i.e. when f_trace is set).  At other times we use
       PyCode_Addr2Line to calculate the line from the current
       bytecode index. */
    int f_lineno;       /* Current line number 当前字节码对应的源代码行*/
    int f_iblock;       /* index in f_blockstack */
    PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
    PyObject *f_localsplus[1];  /* locals+stack, dynamically sized 动态内存,维护(局部变量+cell对象集合+free对象集合+运行时栈)所需要的空间*/
} PyFrameObject;
```

程序运行, 会生成PyFrameObject执行环境(栈帧)链, f_back指向前一个对象

```py
PyFrameObject *
PyFrame_New(PyThreadState *tstate, PyCodeObject *code, PyObject *globals,
            PyObject *locals)
{
...
extras = code->co_stacksize + code->co_nlocals + ncells +
            nfrees; #四部分构成PyFrameObject动态内存区域
...
}
```

##2.2. 名字/作用域和名字空间(重要)

python赋值语句的动作:

1. 创建一个对象
2. 将对象赋给一个名字name(函数, 类定义和import等也赋值语句)

一个约束在程序正文的某个位置是否起作用, 是由约束在文本中的位置是否唯一决定的, 不动态决定的, `所以Python具有静态作用域(编译时就知道名字空间)`, 程序作用域在python运行转化为`名字空间`, python支持嵌套作用域.

- Python最顶层作用域为builtin作用域
- 一个源文件对应一个global作用域
- 每个函数定义一个local作用域
- `LGB查找规则, 现在local作用域查找->再到global作用域查找->最后到python自身定义的builtin作用域查找`
- `LE(enclosing直接外围作用域)GB`(闭包)

```
a = 1
def test():
    print a #1
    a = 2 #2
    print a

test()
#会输出错误:UnboundLocalError: local variable 'a' referenced before assignment
```

原因在注释1和注释2同在一个作用域, 所以在按照LEGB规则, 在local名字空间可以找到名字a, 但是a的赋值发生在注释1之后, 所以出现错误. `global关键字解决这种问题`


##2.3. Python虚拟机的运行框架

python虚拟机的具体实现

```py
#ceval.c
PyObject *
PyEval_EvalFrameEx(PyFrameObject *f, int throwflag
```

- Python进入PyEval_EvalFrameEx函数
- 通过for循环取出字节码,查看对应的参数(如果有的话)
- 结束字节码的执行
- 最后why指示退出for循环时python执行引擎的状态


##2.4. Python运行时环境

- PyFrameObject对应可执行文件执行时的栈帧
- 进程(PyInterpreterState, 线程的活动环境)和线程(PyThreadState对象是线程状态信息的抽象)
- 其中线程同步通过全局解释锁(Global Interpreter Lock)实现

```py
#pystate.c
typedef struct _is {

    struct _is *next;
    struct _ts *tstate_head;  #模拟进程环境中的线程集合

    PyObject *modules;
    PyObject *sysdict;
    PyObject *builtins;
    PyObject *modules_reloading;

    PyObject *codec_search_path;
    PyObject *codec_search_cache;
    PyObject *codec_error_registry;

#ifdef HAVE_DLOPEN
    int dlopenflags;
#endif
#ifdef WITH_TSC
    int tscdump;
#endif

} PyInterpreterState;

typedef struct _ts {
    /* See Python/ceval.c for comments explaining most fields */

    struct _ts *next;
    PyInterpreterState *interp;

    struct _frame *frame;  #模拟线程中的函数调用堆栈, PyFrameObject对象
    int recursion_depth;
    /* 'tracing' keeps track of the execution depth when tracing/profiling.
       This is to prevent the actual trace/profile code from being recorded in
       the trace/profile. */
    int tracing;
    int use_tracing;

    Py_tracefunc c_profilefunc;
    Py_tracefunc c_tracefunc;
    PyObject *c_profileobj;
    PyObject *c_traceobj;

    PyObject *curexc_type;
    PyObject *curexc_value;
    PyObject *curexc_traceback;

    PyObject *exc_type;
    PyObject *exc_value;
    PyObject *exc_traceback;

    PyObject *dict;  /* Stores per-thread state */

    /* tick_counter is incremented whenever the check_interval ticker
     * reaches zero. The purpose is to give a useful measure of the number
     * of interpreted bytecode instructions in a given thread.  This
     * extremely lightweight statistic collector may be of interest to
     * profilers (like psyco.jit()), although nothing in the core uses it.
     */
    int tick_counter;

    int gilstate_counter;

    PyObject *async_exc; /* Asynchronous exception to raise */
    long thread_id; /* Thread id where this tstate was created */

    int trash_delete_nesting;
    PyObject *trash_delete_later;

    /* XXX signal handlers should also be here */

} PyThreadState;
```

python虚拟机开始执行, 会先将PyThreadState对象中的frame设置为当前执行环境(PyFrameObject), 当简称新的PyFrameObject环境时, 线程状态对象取出旧的frame, 建立PyFrameObject链表(在PyFrame_NEW中完成 `frameobject.c`)


> Python编译器完成编译后, 得到PyCodeObject对象,包含源文件的常量表(`co_consts`)和符号表(`co_names`), 然后形成PyFrameObject链, 在PyEval_EvalFrame函数中进行字节码指令执行, why维护python虚拟机中执行字节码执行的状态, WHY_NOT表示正常, WHY_EXCEPTION表示有异常抛出. 处理异常时, 创建traceback对象, 记录异常发生时活动栈帧的状态, 重新设置当前线程状态对象的活动帧, 完成栈帧回退动作, 最后python虚拟机流程一直返回到PyRun_SimpleFileExFlags函数(pythonrun.c), 然后会盗用PyErr_Print函数打印异常.(这样说整个过程不知道对不对, 还需要再读一遍书再来重新整理)

```py
#traceback.h
typedef struct _traceback {
    PyObject_HEAD
    struct _traceback *tb_next;  #链式结构
    struct _frame *tb_frame;  #一个PyTracebackObject对应一个PyFrameObject对象
    int tb_lasti;
    int tb_lineno;
} PyTracebackObject;

#traceback.c, 创建一个traceback对象
int PyTraceBack_Here(PyFrameObject *frame)
{
    #获得线程的状态对象
    PyThreadState *tstate = PyThreadState_GET();
    #保存线程状态对象中维护的traceback对象
    PyTracebackObject *oldtb = (PyTracebackObject *) tstate->curexc_traceback;
    #创建新的traceback对象
    PyTracebackObject *tb = newtracebackobject(oldtb, frame);
    if (tb == NULL)
        return -1;
    #将心的traceback对象交给线程状态对象
    tstate->curexc_traceback = (PyObject *)tb;
    Py_XDECREF(oldtb);
    return 0;
}

#线程traceback对象链表
static PyTracebackObject *
newtracebackobject(PyTracebackObject *next, PyFrameObject *frame)
```

> PyFrameObjectd是对栈帧的模拟， PyFrameObject连接成一条对象链时，就是对栈的模拟


#3. Python虚拟机中的函数机制
---


##3.1. PyFunctionObject对象
---

函数的python对象是PyFunctionObject

```py
#funcobject.h
typedef struct {
    PyObject_HEAD
    PyObject *func_code;    /* A code object 函数编译后的PyCodeObject对象*/
    PyObject *func_globals; /* A dictionary (other mappings won't do) 函数运行时global名字空间*/
    PyObject *func_defaults;    /* NULL or a tuple 默认参数*/
    PyObject *func_closure; /* NULL or a tuple of cell objects 闭包的实现*/
    PyObject *func_doc;     /* The __doc__ attribute, can be anything 函数文档*/
    PyObject *func_name;    /* The __name__ attribute, a string object 函数名称*/
    PyObject *func_dict;    /* The __dict__ attribute, a dict or NULL 函数的__dict__*/
    PyObject *func_weakreflist; /* List of weak references */
    PyObject *func_module;  /* The __module__ attribute, can be anything */

    /* Invariant:
     *     func_closure contains the bindings for func_code->co_freevars, so
     *     PyTuple_Size(func_closure) == PyCode_GetNumFree(func_code)
     *     (func_closure may be NULL if PyCode_GetNumFree(func_code) == 0).
     */
} PyFunctionObject;
```

- PyCodeObject是对源代码的静态表示
- PyFunctionObject是python代码运行时动态产生的(执行def时创建,包含静态信息)

- 无参函数调用

1. 加载函数对应PyCodeObject到运行栈
2. 在执行MAKE_FUCNTION时,将对象弹出栈
3. 通过PyFunction_New创建新的PyFunctionObject对象,并传入源对象的global名称空间
4. 将新创建的对象压入运行栈
5. 执行到函数调用CALL_FUNCTION指令时, 获得栈顶指针然后执行call_function函数
6. 在fast_function函数中进入PyEval_EvalFrameEx,在新的frame栈帧(传入了PyFunctionObject对象中的字节码指令序列PyCodeObject和global名字空间)环境下执行函数中的语句


> PyFunctionObject对象主要是对字节码指令和global名字空间进行打包和运输
> Function, CFunction和Method都会调用这个函数

```py
#ceval.c
static PyObject *
call_function(PyObject ***pp_stack, int oparg
#ifdef WITH_TSC
                , uint64* pintr0, uint64* pintr1
#endif
                )
{
    #1. 处理函数参数信息
    int na = oparg & 0xff;  #oparg记录了参数的个数, na是位置参数的个数
    int nk = (oparg>>8) & 0xff;  #nk是键参数的个数
    int n = na + 2 * nk;
    #2. 获得PyFunctionObject对象
    PyObject **pfunc = (*pp_stack) - n - 1;
    PyObject *func = *pfunc;
    PyObject *x, *w;

    /* Always dispatch PyCFunction first, because these are
       presumed to be the most frequent callable object.
    */
    if (PyCFunction_Check(func) && nk == 0) {
        ...
        #3. 调用PyFunctionObject对象
        if (PyFunction_Check(func)) 
            x = fast_function(func, pp_stack, n, na, nk);  #一般函数通道和快速通道(通过参数形式来确定是否进入快速通道)
        else
            x = do_call(func, pp_stack, na, nk);
        READ_TIMESTAMP(*pintr1);
        Py_DECREF(func);
    }

    /* Clear the stack of the function object.  Also removes
       the arguments in case they weren't consumed already
       (fast_function() and err_args() leave them on the stack).
     */
    while ((*pp_stack) > pfunc) {
        w = EXT_POP(*pp_stack);
        Py_DECREF(w);
        PCALL(PCALL_POP);
    }
    return x;
}
```


##3.2. 参数的传递机制

- 位置参数(positional argument): f(a, b)中的a, b
- 键参数(key argument): f(a, b, name = 'Python')
- 扩展位置参数: f(a, b, *args) 
- 扩展键参数: f(a, b, **kwargs) 

> 函数参数和局部变量关系密切

1. LOAD字节码指令将参数压入运行时栈
2. 调用CALL_FUNCTION, 判断后调用fast_function
3. 拷贝函数参数,从运行时栈到PyFrameObject.f_localplus
4. 参数拷贝结束后, 进入新的PyEval_EvalFrameEx开始真正函数调用
5. 在访问函数参数时,直接通过索引访问f_localsplus中存储的符号对应的值对象
6. 对于带默认参数的函数处理有所不同, 会讲默认参数放入PyFunctionObject.func_defaults, 在fast_function中调用PyEval_EvalCodeEx(ceval.c)


##3.3. 扩展位置参数和扩展键参数

> `*args`和`*kwargs`作为一个局部变量实现, 分别有PyTupleObject和PyDictObject实现

```py
#ceval.c
PyObject *
PyEval_EvalCodeEx(PyCodeObject *co, PyObject *globals, PyObject *locals,
           PyObject **args, int argcount, PyObject **kws, int kwcount,
           PyObject **defs, int defcount, PyObject *closure) #位置参数, 键参数, 默认参
{
    register PyFrameObject *f;
    register PyObject *retval = NULL;
    register PyObject **fastlocals, **freevars;
    PyThreadState *tstate = PyThreadState_GET();
    PyObject *x, *u;

    if (globals == NULL) {
        PyErr_SetString(PyExc_SystemError,
                        "PyEval_EvalCodeEx: NULL globals");
        return NULL;
    }

    assert(tstate != NULL);
    assert(globals != NULL);
    f = PyFrame_New(tstate, co, globals, locals);  #创建PyFrameObject对象
    if (f == NULL)
        return NULL;

    fastlocals = f->f_localsplus;
    freevars = f->f_localsplus + co->co_nlocals;
    #1. 判断是否需要处理扩展位置参数或者扩展键参数
    if (co->co_argcount > 0 ||
        co->co_flags & (CO_VARARGS | CO_VARKEYWORDS)) {
        ...
        }
        ...
        #2. 设置位置参数的参数值
        for (i = 0; i < n; i++) {
            x = args[i];
            Py_INCREF(x);
            SETLOCAL(i, x);
        }
        #3. 处理扩展位置参数
        if (co->co_flags & CO_VARARGS) {
            u = PyTuple_New(argcount - n); #4. 将PyTupleObject对象放入f_localsplus中
            if (u == NULL)
                goto fail;
            SETLOCAL(co->co_argcount, u);
            for (i = n; i < argcount; i++) { #5. 将扩展位置参数放入到PyTupleObject中
                x = args[i];
                Py_INCREF(x);
                PyTuple_SET_ITEM(u, i-n, x);
            }
        }
        for (i = 0; i < kwcount; i++) {
            PyObject **co_varnames;
            PyObject *keyword = kws[2*i];
            PyObject *value = kws[2*i + 1];
            int j;
            if (keyword == NULL || !(PyString_Check(keyword)
#ifdef Py_USING_UNICODE
                                     || PyUnicode_Check(keyword)
#endif
                        )) {
                PyErr_Format(PyExc_TypeError,
                    "%.200s() keywords must be strings",
                    PyString_AsString(co->co_name));
                goto fail;
            }
        ...
}
```


- 当编译函数发现扩展位置参数后, 会在编译得到的PyCodeObject对象添加CO_VARARGS标识
- 对键扩展位置参数, 添加CO_VARKEYWORDS标识, 对扩展X参数放入对应的对象中
- 对象被放到frame中f_localsplus中
- 对于键参数, python虚拟机会先判断是否是一般的键参数


##3.4. 嵌套函数/闭包与装饰器

> 名字空间是在运行时python虚拟机动态维护的, 但时候希望名字空间静态化. 一种方法是将名字空间与函数绑定(`闭包`)

闭包通过嵌套函数实现, PyCodeObject, PyFrameObject有关嵌套函数的属性

```py
#PyCodeObject
co_cellvars  #通常为tuple, 保存嵌套的作用域中使用的变量名集合
co_freevars  #通常为tuple, 保存使用了外层作用域中的变量名集合
#PyFrameObject
f_localsplus #extras = code->co_stacksize + code->co_nlocals + ncells + nfrees; 运行时栈+局部变量+cell对象+free对象
```


闭包执行 :

1. CALL_FUNCTION指令执行fast_funciton, 判断是否进入快速通道
2. python虚拟机创建cell对象(PyCellObject), 将参数(包括cell)拷贝到新创建的PyFrameObject中的f_localsplus
3. 处理cell结束后, 虚拟机进入PyEval_EvalFrameEx,正式对外层函数调用
4. 在执行内部函数时, 虚拟机会将外层的参数对应关系放到PyFunctionObject中.


```py
#cellobject.h
typedef struct {
    PyObject_HEAD
    PyObject *ob_ref;   /* Content of the cell or NULL when empty */
} PyCellObject;

#cellobject.c
PyObject *
PyCell_New(PyObject *obj)
{
    PyCellObject *op;

    op = (PyCellObject *)PyObject_GC_New(PyCellObject, &PyCell_Type);
    if (op == NULL)
        return NULL;
    op->ob_ref = obj;
    Py_XINCREF(obj);

    _PyObject_GC_TRACK(op);
    return (PyObject *)op;
}

#funcobject.h
#define PyFunction_GET_CLOSURE(func) \
    (((PyFunctionObject *)func) -> func_closure)
```


> 装饰器则是对函数的闭包包装模式


#4. Python虚拟机中的类机制
---

##4.1. python中的对象模型

python中三类对象

- type对象:python内置对象的类型
- class对象:python程序员自定义的类型
- instance对象: 表示由class对象创建的实例

> 通过对象`__class__`属性或者python内置type方法可以探测is-instance-of关系(对象与实例), 使用对象的`__base__`属性可以探测is-kind-of关系

- 任何一个class类的type都是metaclass对象(`<type, type>`, 内部为PyType_Type)
- Python中只要一个对象对应的class实现了`__call__`操作,那么这个对象就是可调用对象(在运行期在PyObject_CallFuunctionObjArgs中确定)



python启动时,对类型系统进行初始化, 从`PyType_Ready`开始


```py
#typeobject.c
int
PyType_Ready(PyTypeObject *type)
{
    PyObject *dict, *bases;
    PyTypeObject *base;
    Py_ssize_t i, n;

    if (type->tp_flags & Py_TPFLAGS_READY) {
        assert(type->tp_dict != NULL);
        return 0;
    }
    assert((type->tp_flags & Py_TPFLAGS_READYING) == 0);

    type->tp_flags |= Py_TPFLAGS_READYING;

#ifdef Py_TRACE_REFS
    /* PyType_Ready is the closest thing we have to a choke point
     * for type objects, so is the best place I can think of to try
     * to get type objects into the doubly-linked list of all objects.
     * Still, not all type objects go thru PyType_Ready.
     */
    _Py_AddToAllObjects((PyObject *)type, 0);
#endif

    /* Initialize tp_base (defaults to BaseObject unless that's us) 尝试获得type的tp_base中指定的基类 class对象都是直接或者间接以PyBaseObject_Type(object)为基类 */
    base = type->tp_base;
    if (base == NULL && type != &PyBaseObject_Type) {
        base = type->tp_base = &PyBaseObject_Type;
        Py_INCREF(base);
    }

    /* Now the only way base can still be NULL is if type is
     * &PyBaseObject_Type.
     */

    /* Initialize the base class 如果基类没有初始化, 先初始化基类*/
    if (base && base->tp_dict == NULL) {
        if (PyType_Ready(base) < 0)
            goto error;
    }

    /* Initialize ob_type if NULL.      This means extensions that want to be
       compilable separately on Windows can call PyType_Ready() instead of
       initializing the ob_type field of their type objects. */
    /* The test for base != NULL is really unnecessary, since base is only
       NULL when type is &PyBaseObject_Type, and we know its ob_type is
       not NULL (it's initialized to &PyType_Type).      But coverity doesn't
       know that. 设置基类信息*/
    if (Py_TYPE(type) == NULL && base != NULL)
        Py_TYPE(type) = Py_TYPE(base);
    ...
}
```

- 处理基类和type信息(type中的tp_base中指定基类)
- 处理基类的列表(多重继承)
- 填充tp_dict(将与类型相关的descriptor加入tp_dict中, 其中slot表示PyTypeObject中定义的操作.通过slotdef结构体实现, 在init_slotdefs对整个slotdefs进行排序(操作优先级)), 一个操作对应一个包装slot的PyObject, 称为descriptor
- tp_dict建立从从操作名到descriptor的关联(slot)
- 确定MRO(class对象属性解析顺序)
- 基本mro列表从基类继承操作
- 填充基本中的子类列表


```py
#typeobject.c
typedef struct wrapperbase slotdef;

#descrobject.c
struct wrapperbase {
    char *name;  #操作对应的名字
    int offset;  #操作函数在PyHeapTypeObject中的偏移量
    void *function;  #slot function函数
    wrapperfunc wrapper;
    char *doc;
    int flags;
    PyObject *name_strobj;
};

#创建descriptor的函数
PyObject *
PyDescr_NewWrapper(PyTypeObject *type, struct wrapperbase *base, void *wrapped)
{
    PyWrapperDescrObject *descr;

    descr = (PyWrapperDescrObject *)descr_new(&PyWrapperDescr_Type,
                                             type, base->name);
    if (descr != NULL) {
        descr->d_base = base;
        descr->d_wrapped = wrapped;  #d_wrapped存放操作对应的函数指针
    }
    return (PyObject *)descr;
}
```


##4.2. 用户自定义class及instance对象


- 创建一个class对象, class的动态元信息存放在local名字空间中
- 获得class动态原信息/class名称/class基类列表, 虚拟机调用build_class创建class对象, 然后压入运行栈
- 将创建class对象存入local名称空间
- 创建实例时, 从local名称空间取出对象, 压入运行时栈
- 调用class对象创建instance对象
- 将instance对象存入loacal名字空间
- python虚拟机使用object_new调用object继承来的tp_alloc(PyType_GenericAlllc)申请内存
- 回到type_csll进行初始化(tp_init指向slotdefs中指定与`__init__`对应的`slot_tp_init`)




```py
#typeobject.c
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    PyObject *obj;

    if (type->tp_new == NULL) {
        PyErr_Format(PyExc_TypeError,
                     "cannot create '%.100s' instances",
                     type->tp_name);
        return NULL;
    }

    obj = type->tp_new(type, args, kwds);
    if (obj != NULL) {
        /* Ugly exception: when the call was type(something),
           don't call tp_init on the result. */
        if (type == &PyType_Type &&
            PyTuple_Check(args) && PyTuple_GET_SIZE(args) == 1 &&
            (kwds == NULL ||
             (PyDict_Check(kwds) && PyDict_Size(kwds) == 0)))
            return obj;
        /* If the returned object is not an instance of type,
           it won't be initialized. */
        if (!PyType_IsSubtype(obj->ob_type, type))
            return obj;
        type = obj->ob_type;
        if (PyType_HasFeature(type, Py_TPFLAGS_HAVE_CLASS) &&
            type->tp_init != NULL &&
            type->tp_init(obj, args, kwds) < 0) {
            Py_DECREF(obj);
            obj = NULL;
        }
    }
    return obj;
}
```


##4.3. 访问instance对象中属性

形如`x.y或x.y()`称为属性引用.

- instance对象入栈
- 通过PyObject_GetAttr获得属性

```py
#object.c
PyObject *
PyObject_GetAttr(PyObject *v, PyObject *name)
{
    PyTypeObject *tp = Py_TYPE(v);

    if (!PyString_Check(name)) {
#ifdef Py_USING_UNICODE
        /* The Unicode to string conversion is done here because the
           existing tp_getattro slots expect a string object as name
           and we wouldn't want to break those. */
        if (PyUnicode_Check(name)) {
            name = _PyUnicode_AsDefaultEncodedString(name, NULL);
            if (name == NULL)
                return NULL;
        }
        else
#endif
        {
            PyErr_Format(PyExc_TypeError,
                         "attribute name must be string, not '%.200s'",
                         Py_TYPE(name)->tp_name);
            return NULL;
        }
    }
    #1. 通过tp_getattro获得属性对应对象
    if (tp->tp_getattro != NULL)
        return (*tp->tp_getattro)(v, name);
    #2. 通过tp_getattr获得属性对应对象
    if (tp->tp_getattr != NULL)
        return (*tp->tp_getattr)(v, PyString_AS_STRING(name));
    #3. 属性不存在,抛出异常
    PyErr_Format(PyExc_AttributeError,
                 "'%.50s' object has no attribute '%.400s'",
                 tp->tp_name, PyString_AS_STRING(name));
    return NULL;
}
```
