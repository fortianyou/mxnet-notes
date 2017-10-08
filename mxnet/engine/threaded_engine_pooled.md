ThreadEnginePooled
=====

ThreadPool
----

### 构造函数

`explict ThreadPool(size_t size, std::function<void> func);`

+ size: 线程池大小
+ func: 每个线程(std::thread)运行的函数, 通常不会退出.

`explict ThreadPool(size_t size, std::function<void> func, bool wait);`

### SimpleEvent

在表销毁时一定能被 signal 的事件.

ThreadEnginePooled
---

ThreadEngine using global thread pool across all devices.
The policy of this Engine.
+ Exec Async operation immediately if pushed from Pusher.
+ Use a common thread pool for normal operations on all devices.
+ Use special thread pool for copy operations.

### 构造函数

创建了 1 个具有 kNumWorkingThreads 的工作线程池.
创建了 1 个具有 1 个线程的拷贝线程池.

每个线程启动一个 ThreadWorker 消费者.

### 调度机制

拥有两个队列: 
`task_queue` 和 `io_task_queue` 

### override

`void PushToExecute(OprBlock *opr_block, bool pusher_thread);`

+ Exec Async operation immediately if pushed from Pusher, use `DoExecute`
+ otherwise, enqueue the opr_block, use `DoPushToQueue`

### 内部实现

`void ThreadWorker(dmlc::ConcurrentBlockingQueue<OprBlock*>* task_queue);`

+ task_queue: 是一个 `dmlc::ConcurrentBlockingQueue<OprBlock*>` 类型的队列.
+ 生产者消费者模式: ThreadWorker 是一个消费者, 使用`DoExecute`处理从队列中取出 `OprBlock`.

`void DoExecute(OprBlock* opr_block);`

+ 设置设备(GPU)
+ 获取RunContext, IORunContext/RunContext
+ ThreadEngine::ExecuteOprBlock

`void DoPushToQueue(OprBlock* opr_block);`
+ 根据 opr_block::prop 确定加入 `io_task_queue/task_queue`