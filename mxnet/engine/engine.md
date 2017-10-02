Engine
====

mxnet/engine不仅可以用于深度学习框架, 还可以应用在任意按照依赖关系执行的函数调度中.

+ 对于存在依赖关系的函数, 严格按照串行顺序执行
+ 对于没有依赖关系的函数, 为了提升性能可以被并行调度.

Interface
----

Engine 的核心接口如下所示:

```
virtual void PushSync(Fn exec_fun, Context exec_ctx,
                          std::vector<VarHandle> const& const_vars,
                          std::vector<VarHandle> const& mutate_vars) = 0;
```

上面 API 允许通过`PushSync`方法向 Engine 发送一个带有依赖关系的执行函数`exec_func`.
其中, `exec_ctx`表示执行`exec_func`所需的环境, `const_vars` 和 `mutate_vars`表示读入和写入的变量.

**NOTE**: Engine保证任何两个修改同一个变量的执行函数都将会按照其`push`的顺序被调度.

Function
-----

Engine执行的函数类型如下:

```
using Fn = std::function<void(RunContext)>;
struct RunContext {
   // stream pointer which could be safely cast to
   // cudaStream_t* type
   void *stream;
;
``` 

所有`Fn`都在 Engine 内部线程中运行. 在这种情况下, 如果`push`了一个阻塞的`Fn`, 当函数被调用时同样会阻塞Engine线程.
为了提高 Engine 的吞吐量和性能, Engine 提供了异步函数调用:

```
using Callback = std::function<void()>;
using AsyncFn = std::function<void(RunContext, Callback)>;
```
按照这种方式, 应将较耗时的步骤应该放在自己线程中执行. 由`AsyncFunc`启动该线程并安全地返回, 直到`Callback`函数被调用, Engine 才会认为`AsyncFunc`执行完毕.

Context
----

Context 表示了执行`Fn`所需的运行环境(GPU/CPU, 设备 id). RunContext 表示运行环境, 比如使用哪个 stream.

VarHandle
----

VarHandle 用于确定`Fn`的依赖. Mxnet Engine 设计之初, 便于其他Mxnet 模块解耦. VarHandle 可以看做Mxnet Engine 对外提供的用于持有外部资源的token. 借助 VarHandle 在 Engine 内部`Fn`可以使用和修改外部资源.

VarHandle 的实现是轻量级的, 所以拷贝, 创建, 删除一个变量几乎不花费资源.
 在执行`push`之前, 需要确定哪些变量是不可变的, 哪些变量是可变的.

Push and Wait
------

所有Push API 都是异步的调用. Push 不会等待`Fn`执行完毕便返回, `Fn`的执行由 Engine 来调度.
Push 不是线程安全的, 因此需要保证任意时刻只有一个API 调用.

如果希望等待一个特定的 `Fn`执行完成, 则需要在在 `Fn`函数的最后加入 `Callback`函数.

如果希望等待所有使用了某个`Var`的 `Fn`调用执行完毕, 则需要调用 `WaitForVar(var)`.

如果希望等待所有 `Fn`调用执行完毕, 则执行 `WaitForAll()`.

避免频繁调用和创建带来的消耗
-----

在有些情况下, 会频繁并持续地向 Engine Push 函数调用. 
这种情况下如果计算本身时间很少, 则调用过程中的参数拷贝和创建会消耗大部分时间.
此时可以将函数调用封装成一个 OprHandle:

```
virtual OprHandle NewOperator(AsyncFn fn,
                                  std::vector<VarHandle> const& const_vars,
                                  std::vector<VarHandle> const& mutate_vars) = 0;
```

这样子便可以反复向 Engine Push 同一个 OprHandle, 从而避免反复发生参数拷贝和创建.

```
 virtual void Push(OprHandle op, Context exec_ctx) = 0;
```
并使用`DeleteOperator`来删除这个 OprHandle.