title: Python源码剖析读书笔记(三)
date: 2015-01-29 09:25:26
tags: [Python, 读书笔记]
---

> 整本源码剖析的书基本看完, 不过我觉得第二部分有太多不解, 另外第三部分也是精华, 等过段时间会再看一遍, 听说这本书第二版在编辑中, 等书出了一定要第一时间买一本...

#1. Python运行环境初始化
---


python运行时环境初始化从`Py_InitializeEX`(pythonrun.c)开始.其中PyInterpreterState是对进程的模拟, PyThreadState是对线程的模拟.


**Py_InitializeEX:**

<!--more-->

- 调用PyInterpreterState_New创建一个新的PyInterpreterState对象(pystate.c), python运行中interp_head全局管理一个PyInterpreterState对象链表(多进程)
- 调用PyThreadState_New创建一个新的PyThreadState对象(pystate.c),并关联进程和线程
- python运行环境中, _PyThreadState_Current全局变量维护当前活动线程对应的PyThreadState对象
- 创建进程线程对象后, 通过`_PyBuiltin_Init`设置系统`__builtin__` module(所有类型对象,函数集合, 文档),interp->builtins
- 设置sys module, interp->sysdict
- 设置python收缩module的默认搜索路径集合PySys_SetPath(sysmodule.c)
- 初始化import环境`_PyImport_Init()`
- 初始化python内建exceptions
- 备份exceptions module和内建模块
- 在sys module中添加一些对象, 用于import机制
- python之后将创建`__main__` module, 加入到interp->modules(作为主程序运行的源文件可视为`__main__`), python运行程序沿着名字空间找到`__name__`就是`__main__`
- `site.py`将site-package加入到sys.path中


![进程](http://picturebag.qiniudn.com/69.png)

![初始化](http://picturebag.qiniudn.com/70.png)


##1.1. 激活虚拟机

Py_InitializeEX成功完成后, 调用`PyRun_AnyFileExFlags`(main.c)`进入字节码虚拟机, 通过Py_FdIsInteractive判断是否为交互式方式运行, 是则执行PyRun_InteractiveLoopFlags; 不是则执行PyRun_SimpleFileExFlags

- 交互式运行方法
    1. 执行PyRun_InteractiveLoopFlags(pythonrun.c)
    2. 创建交互式环境提示符
    3. 进入交互式环境
    4. 通过PyRun_InteractiveOneFlags函数编译交互式环境输入的python语句(生成抽象语法树), 执行输入的python代码(run_mod)
    

- 脚本文件运行方法
    1. 执行PyRun_SimpleFileExFlags(pythonrun.c)
    2. 在`__main__` module中设置`__file__`属性
    3. 通过PyRun_FileExFlags执行脚本文件
    4. 编译并执行(run_mod)


##1.2. 启动虚拟机

run_mode开始, python启动字节码虚拟机

1. 基于抽象语法树编译字节码指令序列, 创建PycodeObject对象
2. 创建PyFrameObject对象(运行环境), 执行PyCodeObject中的字节码指令序列

```py
#pythonrun.c
static PyObject *
run_mod(mod_ty mod, const char *filename, PyObject *globals, PyObject *locals,
         PyCompilerFlags *flags, PyArena *arena)
{
    PyCodeObject *co;
    PyObject *v;
    co = PyAST_Compile(mod, filename, flags, arena); #编译
    if (co == NULL)
        return NULL;
    #创建PyFrameObject对象, 执行code对象中字节码指令序列
    v = PyEval_EvalCode(co, globals, locals); 
    Py_DECREF(co);
    return v;
}

#ceval.c
PyObject *
PyEval_EvalCode(PyCodeObject *co, PyObject *globals, PyObject *locals)
{   #唤醒字节码虚拟机, 在其中的PyFrame_New设置local/global/builtin名字空间
    return PyEval_EvalCodeEx(co,
                      globals, locals,
                      (PyObject **)NULL, 0,
                      (PyObject **)NULL, 0,
                      (PyObject **)NULL, 0,
                      NULL);
}
```

> python所有的线程共享相同的builtin名字空间

![运行时](http://picturebag.qiniudn.com/65.png)

#2. Python模块的动态加载机制
---

python的一个module从硬盘到内存的加载


```py
def show_moudules() :
    import sys
    for item in sys.modules.items() :
        print item

if __name__ == '__main__':
    show_moudules()
```

嵌套的import操作并不影响上一层的孔子空间, 只影响各自module自身的名字空间(module自身维护的dict)

- Python中,module是由一个单独的文件实现的
- package是通过文件夹来实现的(文件夹中要由`__init__.py`)

**动态加载**

在加载module时, 如A.B.c, python内部将这个module表示为一个树状结构, python动态加载时, 首先会将树状结构分解, 形成(A, B, c)节点集合, 然后从左到右依次到sys.modules中查找每一个符号所对应的module是否已经能够被加载. 如果被加载了, 则A对应PyModuleObject对象中维护着一个元信息`__path__`, 表示这个package路径.此时B,c没有加载, 接下来对B的搜索只在`A.__path__中进行`


> 其中sys.modules集合作为module pool, 保存module的唯一映像, 某个.py文件通过import声明时,python将在这个pool中寻找, 如存在, 则引入符号到该.py的命名空间中, 并将其关联到该module. 如果不在pool, 这是才进行动态加载


##2.1. from和import

from和import结合, 使用python虚拟街在当前名字空间中引入符号尽可能少, 从而避免名字空间污染

**python还提供了符号重命名机制**

通过as关键字控制module以什么名字被引入到当前的local名字空间


**符号的销毁和重载**

通过import加载module会一直占用内存, 通过del在命名空间删除该符号, 通过reload对module进行重载


##2.2. import实现机制

> import机制起点是builtin module中的`___import__`操作(`builtin___import__`函数)

import机制3个功能:

- python运行时的全局module pool的维护和搜索
- 解析与搜索module路径的树状结构
- 对不同文件格式的module的动态加载机制

```c
//bltinmodule.c
static PyObject *
builtin___import__(PyObject *self, PyObject *args, PyObject *kwds)
{
    static char *kwlist[] = {"name", "globals", "locals", "fromlist",
                             "level", 0};
    char *name;
    PyObject *globals = NULL;
    PyObject *locals = NULL;
    PyObject *fromlist = NULL;
    int level = -1;
    //从tuple中解析需要的信息
    if (!PyArg_ParseTupleAndKeywords(args, kwds, "s|OOOi:__import__",
                    kwlist, &name, &globals, &locals, &fromlist, &level))
        return NULL;
    return PyImport_ImportModuleLevel(name, globals, locals,
                                      fromlist, level);
}

#import.c
PyObject *
PyImport_ImportModuleLevel(char *name, PyObject *globals, PyObject *locals,
                         PyObject *fromlist, int level)
{
    PyObject *result;
    _PyImport_AcquireLock();
    result = import_module_level(name, globals, locals, fromlist, level);
    if (_PyImport_ReleaseLock() < 0) {
        Py_XDECREF(result);
        PyErr_SetString(PyExc_RuntimeError,
                        "not holding the import lock");
        return NULL;
    }
    return result;
}
```

1. 执行`builtin___import__`函数完成对参数的拆包(进行import前, 会对import动作加锁, 进入import后, 解锁)
2. 进入PyImport_ImportModuleLevel函数(`import.c`)执行`import_module_level`(对树形结构遍历), 获得import动作发生的packge环境, 解析module的路径结构(树), 依次加载每个package/module
3. 在`import_module_level`中的load_next中调用`import_submodule`完成搜索module并加载module的工作
4. 对于py文件虚拟机执行`load_source_module`(编译, 产生PyCodeObject对象执行字节码); 对于pyc, 虚拟机调用`load_compiled_module`(至少了编译动作); 对于内建module, 只有一部分常用的在运行时被加载到sys.modules中, 对于不常用的进行import操作,, 虚拟机调用`init_builtin`加载module(在内建module列表中找到后, 调用init初始化); 对于C扩展module, 会调用`_PyImport_LoadDynamicModule`(查找python的module备份是否有module, 获得module初始化起始地址并调用, 从sys.modules获得已经加载的module并设置`__file__`属性, 最后加入到python的module备份)

> import动作是发生在一个package环境中


```c
//内建module的完整列表

//import.h
struct _inittab {
    char *name;
    void (*initfunc)(void);
};

//import.c
struct _inittab *PyImport_Inittab = _PyImport_Inittab;

//PC/config.c
struct _inittab _PyImport_Inittab[] = {

    {"array", initarray},
    {"_ast", init_ast},
#ifdef MS_WINDOWS
#ifndef MS_WINI64
    {"audioop", initaudioop},
#endif
#endif
    {"binascii", initbinascii},
    {"cmath", initcmath},
    {"errno", initerrno},
    {"future_builtins", initfuture_builtins},
    {"gc", initgc},
#ifndef MS_WINI64
    {"imageop", initimageop},
#endif
    {"math", initmath},
    {"_md5", init_md5},
    {"nt", initnt}, /* Use the NT os functions, not posix */
    {"operator", initoperator},
    {"signal", initsignal},
    {"_sha", init_sha},
    {"_sha256", init_sha256},
    {"_sha512", init_sha512},
    {"strop", initstrop},
    {"time", inittime},
#ifdef WITH_THREAD
    {"thread", initthread},
#endif
    {"cStringIO", initcStringIO},
    {"cPickle", initcPickle},
#ifdef WIN32
    {"msvcrt", initmsvcrt},
    {"_locale", init_locale},
#endif
    /* XXX Should _subprocess go in a WIN32 block?  not WIN64? */
    {"_subprocess", init_subprocess},

    {"_codecs", init_codecs},
    {"_weakref", init_weakref},
    {"_hotshot", init_hotshot},
    {"_random", init_random},
    {"_bisect", init_bisect},
    {"_heapq", init_heapq},
    {"_lsprof", init_lsprof},
    {"itertools", inititertools},
    {"_collections", init_collections},
    {"_symtable", init_symtable},
    {"mmap", initmmap},
    {"_csv", init_csv},
    {"_sre", init_sre},
    {"parser", initparser},
    {"_winreg", init_winreg},
    {"_struct", init_struct},
    {"datetime", initdatetime},
    {"_functools", init_functools},
    {"_json", init_json},

    {"xxsubtype", initxxsubtype},
    {"zipimport", initzipimport},
    {"zlib", initzlib},

    /* CJK codecs */
    {"_multibytecodec", init_multibytecodec},
    {"_codecs_cn", init_codecs_cn},
    {"_codecs_hk", init_codecs_hk},
    {"_codecs_iso2022", init_codecs_iso2022},
    {"_codecs_jp", init_codecs_jp},
    {"_codecs_kr", init_codecs_kr},
    {"_codecs_tw", init_codecs_tw},

/* tools/freeze/makeconfig.py marker for additional "_inittab" entries */
/* -- ADDMODULE MARKER 2 -- */

    /* This module "lives in" with marshal.c */
    {"marshal", PyMarshal_Init},

    /* This lives it with import.c */
    {"imp", initimp},

    /* These entries are here for sys.builtin_module_names */
    {"__main__", NULL},
    {"__builtin__", NULL},
    {"sys", NULL},
    {"exceptions", NULL},
    {"_warnings", _PyWarnings_Init},

    {"_io", init_io},

    /* Sentinel */
    {0, 0}
};
```


#3. python多线程机制
---

> Python通过GIL来互斥不同线程对解释器的使用, python借助底层操作系统提供的线程调度机制来决定下一个进入python解释器的线程.

##3.1. python thread

```c
//threadmodule.c
static PyMethodDef thread_methods[] = {
    {"start_new_thread",        (PyCFunction)thread_PyThread_start_new_thread,
                            METH_VARARGS,
                            start_new_doc},
    {"start_new",               (PyCFunction)thread_PyThread_start_new_thread,
                            METH_VARARGS,
                            start_new_doc},
    {"allocate_lock",           (PyCFunction)thread_PyThread_allocate_lock,
     METH_NOARGS, allocate_doc},
    {"allocate",                (PyCFunction)thread_PyThread_allocate_lock,
     METH_NOARGS, allocate_doc},
    {"exit_thread",             (PyCFunction)thread_PyThread_exit_thread,
     METH_NOARGS, exit_doc},
    {"exit",                    (PyCFunction)thread_PyThread_exit_thread,
     METH_NOARGS, exit_doc},
    {"interrupt_main",          (PyCFunction)thread_PyThread_interrupt_main,
     METH_NOARGS, interrupt_doc},
    {"get_ident",               (PyCFunction)thread_get_ident,
     METH_NOARGS, get_ident_doc},
    {"_count",                  (PyCFunction)thread__count,
     METH_NOARGS, _count_doc},
    {"stack_size",              (PyCFunction)thread_stack_size,
                            METH_VARARGS,
                            stack_size_doc},
    {NULL,                      NULL}           /* sentinel */
};

static PyObject *
thread_PyThread_start_new_thread(PyObject *self, PyObject *fargs)
{
    PyObject *func, *args, *keyw = NULL;
    struct bootstate *boot;
    long ident;

    if (!PyArg_UnpackTuple(fargs, "start_new_thread", 2, 3,
                           &func, &args, &keyw))
        return NULL;
    if (!PyCallable_Check(func)) {
        PyErr_SetString(PyExc_TypeError,
                        "first arg must be callable");
        return NULL;
    }
    if (!PyTuple_Check(args)) {
        PyErr_SetString(PyExc_TypeError,
                        "2nd arg must be a tuple");
        return NULL;
    }
    if (keyw != NULL && !PyDict_Check(keyw)) {
        PyErr_SetString(PyExc_TypeError,
                        "optional 3rd arg must be a dictionary");
        return NULL;
    }
    //1. 创建并初始化bootstate结构, 在boot中保存关于线程的一切信息, 
    boot = PyMem_NEW(struct bootstate, 1);
    if (boot == NULL)
        return PyErr_NoMemory();
    boot->interp = PyThreadState_GET()->interp;  //保存PyInterpreterState对象
    boot->func = func;
    boot->args = args;
    boot->keyw = keyw;
    boot->tstate = _PyThreadState_Prealloc(boot->interp);
    if (boot->tstate == NULL) {
        PyMem_DEL(boot);
        return PyErr_NoMemory();
    }
    Py_INCREF(func);
    Py_INCREF(args);
    Py_XINCREF(keyw);
    //2. 初始化多线程环境
    PyEval_InitThreads(); /* Start the interpreter's thread-awareness */
    //3. 创建操作系统原生线程
    ident = PyThread_start_new_thread(t_bootstrap, (void*) boot);
    if (ident == -1) {
        PyErr_SetString(ThreadError, "can't start new thread");
        Py_DECREF(func);
        Py_DECREF(args);
        Py_XDECREF(keyw);
        PyThreadState_Clear(boot->tstate);
        PyMem_DEL(boot);
        return NULL;
    }
    return PyInt_FromLong(ident);
}

//pythread.h
typedef void *PyThread_type_lock;

//ceval.c 初始化
static PyThread_type_lock interpreter_lock = 0; /* This is the GIL */
static PyThread_type_lock pending_lock = 0; /* for pending calls */
static long main_thread = 0;
void
PyEval_InitThreads(void)
{
    if (interpreter_lock)
        return;
    interpreter_lock = PyThread_allocate_lock(); //创建GIL(PNRMUTEX aLock)thread_nt.h
    PyThread_acquire_lock(interpreter_lock, 1);
    main_thread = PyThread_get_thread_ident();
}

//thread_nt.c
typedef struct NRMUTEX {
    LONG   owned ;
    DWORD  thread_id ;
    HANDLE hevent ; //event内核对象
} NRMUTEX, *PNRMUTEX ;

```

- 初始化线程环境: PyEval_InitThreads通过PyThread_allocate_lock创建GIL后, 调用任何python C API前, 必须获得GIL(PyThread_acquire_lock), 获得当前主线程的id, 并将其赋值为main_thread
- 创建线程: 主线程为执行程序时操作系统创建,主线程执行初始化获得GIL 子线程是在PyThread_start_new_thread函数中CreateThread创建出来的(程序中的start_new_thread, 是在主线程中执行的). 子线程创建自身线程状态对象后, 通过`_PyGILState_NotrThreadState`将这个对象放入线程状态对象链表


```c
//thread_nt.c 创建线程
long PyThread_start_new_thread(void (*func)(void *), void *arg)
{
    HANDLE hThread;
    unsigned threadID;
    callobj *obj;

    dprintf(("%ld: PyThread_start_new_thread called\n",
             PyThread_get_thread_ident()));
    if (!initialized)
        PyThread_init_thread();

    obj = (callobj*)HeapAlloc(GetProcessHeap(), 0, sizeof(*obj));
    if (!obj)
        return -1;
    obj->func = func;
    obj->arg = arg;
#if defined(MS_WINCE)
    hThread = CreateThread(NULL,
                           Py_SAFE_DOWNCAST(_pythread_stacksize, Py_ssize_t, SIZE_T),
                           bootstrap, obj, 0, &threadID); //boostrap完成获得线程id, 通知obj->done内核对象, 调用t_bootstrap(在这个函数中与主线程竞争GIL, threadmodule.c), 在子线程中执行
#else
    hThread = (HANDLE)_beginthreadex(0,
                      Py_SAFE_DOWNCAST(_pythread_stacksize,
                                       Py_ssize_t, unsigned int),
                      bootstrap, obj,
                      0, &threadID);
#endif
    if (hThread == 0) {
#if defined(MS_WINCE)
        /* Save error in variable, to prevent PyThread_get_thread_ident
           from clobbering it. */
        unsigned e = GetLastError();
        dprintf(("%ld: PyThread_start_new_thread failed, win32 error code %u\n",
                 PyThread_get_thread_ident(), e));
#else
        /* I've seen errno == EAGAIN here, which means "there are
         * too many threads".
         */
        int e = errno;
        dprintf(("%ld: PyThread_start_new_thread failed, errno %d\n",
                 PyThread_get_thread_ident(), e));
#endif
        threadID = (unsigned)-1;
        HeapFree(GetProcessHeap(), 0, obj);
    }
    else {
        dprintf(("%ld: PyThread_start_new_thread succeeded: %p\n",
                 PyThread_get_thread_ident(), (void*)hThread));
        CloseHandle(hThread);
    }
    return (long) threadID;
}
```


![线程](http://picturebag.qiniudn.com/71.png)


每个将要从运行态转为等待态的线程都会在被挂起前调用PyThread_release_lock释放GIL(thread_xx.h)并通知所有等待GIL `event内核对象`的线程


##3.2. python线程的调度

> python线程调度机制内建在python解释器的核心PyEval_EvalFrameEx中


主线程先获得GIL, 并执行PyEval_EvalFrameEx函数代码, 这是子线程在t_bootstrap中调用PyEval_AcquireThread, 通过调用PyThread_acquire_lock申请GIL, 但由于GIL被主线程调用, 子线程被挂起. 主线程不断执行字节码, `_Py_Ticker`不断减一, 当减到0, 将当前维护线程状态置NULL, 然后释放GIL,此时子线程`被操作系统的线程调度唤醒`, 从而进入PyEval_EvalFrameEx. 对于主线程虽然失去了GIL, 但是没被挂起, 所以可以被再次切换为活动线程, 再次申请GIL, 由于被子线程占有, 主线程将自身挂起.


##3.3. 阻塞调度

线程A通过某些操作(如等待输入), 将自身阻塞, python应将等待GIL的线程B唤醒.

##3.4. python子线程的销毁

> 主线程销毁必须要销毁python的运行时环境, 子线程的销毁不需要进行这些动作

线程的主框架在`t_bootstrap`中,

```c
//threadmodule.c
static void
t_bootstrap(void *boot_raw)
{
    struct bootstate *boot = (struct bootstate *) boot_raw;
    PyThreadState *tstate;
    PyObject *res;

    tstate = boot->tstate;
    tstate->thread_id = PyThread_get_thread_ident();
    _PyThreadState_Init(tstate);
    PyEval_AcquireThread(tstate);
    nb_threads++;
    res = PyEval_CallObjectWithKeywords(
        boot->func, boot->args, boot->keyw);
    if (res == NULL) {
        if (PyErr_ExceptionMatches(PyExc_SystemExit))
            PyErr_Clear();
        else {
            PyObject *file;
            PyObject *exc, *value, *tb;
            PyErr_Fetch(&exc, &value, &tb);
            PySys_WriteStderr(
                "Unhandled exception in thread started by ");
            file = PySys_GetObject("stderr");
            if (file)
                PyFile_WriteObject(boot->func, file, 0);
            else
                PyObject_Print(boot->func, stderr, 0);
            PySys_WriteStderr("\n");
            PyErr_Restore(exc, value, tb);
            PyErr_PrintEx(0);
        }
    }
    else
        Py_DECREF(res);
    Py_DECREF(boot->func);
    Py_DECREF(boot->args);
    Py_XDECREF(boot->keyw);
    PyMem_DEL(boot_raw);
    nb_threads--;
    PyThreadState_Clear(tstate);  //清理当前线程锁对应的线程状态对象, 引用计数的维护
    PyThreadState_DeleteCurrent();  //释放GIL
    PyThread_exit_thread();  //完成不同平台销毁原生线程的工作
}
```

##3.5. Python线程的用户级互斥和同步

> python的线程在GIL控制下, 线程之间, 对python解释器, 对C API的访问都是互斥的.(`内核级互斥, 用户不可控`)

thread中提供了Lock对象, thread.allocate对应的C函数`thread_PyThread_allocate_lock`, 创建一个lockobject对象,


```c
//threadmodule.c
/* Lock objects */
typedef struct {
    PyObject_HEAD
    PyThread_type_lock lock_lock;  //Event内核对象
    PyObject *in_weakreflist;
} lockobject;

static PyObject *
thread_PyThread_allocate_lock(PyObject *self)
{
    return (PyObject *) newlockobject();
}
static lockobject *
newlockobject(void)
{
    lockobject *self;
    self = PyObject_New(lockobject, &Locktype);
    if (self == NULL)
        return NULL;
    self->lock_lock = PyThread_allocate_lock();
    self->in_weakreflist = NULL;
    if (self->lock_lock == NULL) {
        Py_DECREF(self);
        PyErr_SetString(ThreadError, "can't allocate lock");
        return NULL;
    }
    return self;
}
```

关于对thread module封装的threading, 调用threading.thread.start时, 会在`_limbo`中记录线程, 通过thread.start_new_thread创建原生线程, 线程过程为`__bootstrap`. 在bootstrap中, 会在_limo中删除线程记录, 转而将线程记录到`_active`中, 然后调用run, run结束后, 会调用__stop操作, 会调用其中Condition对象的notifyAll通知等待该对象的线程, 这些线程会通过threading.Thread.join操作注册成为Condition对象的等待线程.


#4. Python内存管理
---


> python所有内存管理有两套实现, 由编译符号`PYMALLOC_DEBUG控制`, debug模式和正常的内存管理模式

内存管理的抽象层次:

1. 最底层(第0层), 操作系统提供的内存管理接口, 比如C运行时的malloc和free接口, 由操作系统管理和实现, python不能干涉, 其他层由python实现并维护
2. 第1层是python基于第0层操作系统的内存管理接口包装形成的(处理平台相关的内存分配行为, PyMem_API)
3. 第2层更高抽象层次的内存管理接口, 是一组以PyObje_为前缀的函数族(主要提供创建Python对象的接口)
4. 第3层主要是对对象缓冲池机制(GC机制)

##4.1. 小块空间的内存池

> 小块内存池可以看做4层结构: `block, pool, arena, 内存池`

- block是确定大小的内存块(8字节对齐, 8, 16, 32...512, 内存小大称为size class, 每个size class对应一个size class index, index从0开始)

```c
//obmalloc.c
#define ALIGNMENT               8               /* must be 2^N */
#define ALIGNMENT_SHIFT         3
#define ALIGNMENT_MASK          (ALIGNMENT - 1)

#define SMALL_REQUEST_THRESHOLD 512 
#define NB_SMALL_SIZE_CLASSES   (SMALL_REQUEST_THRESHOLD / ALIGNMENT)
```

> 当申请一个28字节内存时, PyObject_Malloc从内存池中分配一个32字节的block, 从size class index为3的pool中分配

- pool管理一堆固定大小的内存块, pool大小一般为系统内存页(一般为4K, 包括pool_header头部和block几何占用的内存) 


```c
//obmalloc.c
#define SYSTEM_PAGE_SIZE        (4 * 1024)
#define SYSTEM_PAGE_SIZE_MASK   (SYSTEM_PAGE_SIZE - 1)
#define POOL_SIZE               SYSTEM_PAGE_SIZE        /* must be 2^N */
#define POOL_SIZE_MASK          SYSTEM_PAGE_SIZE_MASK
```

> 每一个pool都和一个size class index关联.

将4K内存转换为pool, pool先指向一块4K内存, 设置pool的size class index, 将index转换为size(index = 3即32字节, index = 0即8字节),跳过pool_header的内存, 进行对齐, 当对其中block进行分配时, 如果某些block内存被释放了, 则通过自由block链表(pool_header中的freeblock)进行管理


- arena是多个pool的集合(256KB), 容纳pool的个数是ARENA_SIZE/POOL_SIZE = 64, `arena管理的内存是分离的, pool_header自身是连续的内存`

```c
//obmalloc.c
#define ARENA_SIZE              (256 << 10)     /* 256KB */
struct arena_object {
    /* The address of the arena, as returned by malloc.  Note that 0
     * will never be returned by a successful malloc, and is used
     * here to mark an arena_object that doesn't correspond to an
     * allocated arena.
     */
     ...
     }
```

当arena的area_object没和poll集合建立联系, arena处于未使用状态(单链表头`unused_arena_objects`), 一旦建立联系, arena转换为可用状态(双向链表头`usable_arenas`), 每个状态有一个arena链表

> arena是小块内存池的最上层结构, 所有arena的集合是小块内存池

(小块内存和大块内存分界点是512字节), 小于512会在内存池中申请, 申请内存大于512时, PyObject_Malloc蜕变为molloc行为. python启动后, usedpools这个小块空间内存池并不存在任何可用的内存(不存在可用的pool), python采用延迟分配策略, 当确实开始申请小块内存是, python才建立内存池.


##4.2. 循环引用的垃圾收集

> python中大多数对象的生命周期是通过对象的引用计数来管理的. 广义上, 引用计数是一种最直接, 最简单的垃圾收集计数. python引入了标记-清除和分代收集计数来填补循环引用的缺陷.

垃圾收集机制一般分为两个阶段: 垃圾检测和垃圾回收

- 三色标记模型
    1. 寻找根对象的集合, root object是一些全局引用和函数栈中的引用, 这些引用所用的对象不可被删除, root object集合是垃圾检测的起点
    2. 从root object集合出发, 沿着root object集合中的每个引用, 如果能到达某对象A, 称A可达, 可达对象不能被删除, 这阶段是垃圾检测阶段
    3. 垃圾检测后, 所有对象分为可达和不可达, 可达对象被保留, 不可达对象所占用内存将被回收, 这短短是垃圾回收阶段.

##4.3. Python中的垃圾收集

循环引用总是发生在container之间(内部可持有对其他对象的引用的对象, 如list, dict, class), python垃圾收集只需要检测这些container, container要成为可收集对象, 在PyObject_Head前加入PyGC_Head, 并将可收集container对象连接到python内部维护的可收集对象链表中.

![收集对象](http://picturebag.qiniudn.com/72.png)


- 分代的垃圾收集

将系统所有的内存块按其存活时间划分为不同集合, 每个集合成为一个`代`, 垃圾收集的频率随着`代`的增大而减小. python中的分代收集, 分为`三代`.



- 标记-清除方法

从root object触发前, 将内存链表分为root链表和unreachable链表, 完成可达和不可达的标记, 最后unreachable链表中对象就是垃圾收集对象.


