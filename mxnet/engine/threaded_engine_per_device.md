ThreadedEnginePerDevice
=====

ThreadedEngine uses per device threads

The policy of this ThreadedEngine:

- Exec Aync operation immediately if pushed from Pusher.
- Use fixed amount of threads for each device.
- Use special threads for copy operations.
- Each stream is allocated and bound to each of the thread.

### 内部数据结构

```
// cpu normal worker
common::LazyAllocArray<ThreadWorkerBlock<kWorkerQueue> > cpu_normal_workers_;
// cpu priority worker
std::unique_ptr<ThreadWorkerBlock<kPriorityQueue> > cpu_priority_worker_;
// workers doing normal works on GPU
common::LazyAllocArray<ThreadWorkerBlock<kWorkerQueue> > gpu_normal_workers_;
// workers doing copy works from/to GPU
common::LazyAllocArray<ThreadWorkerBlock<kCopyQueue> > gpu_copy_workers_;
```
CPU 上有两类工作队列: 普通工作队列和优先工作队列.
GPU 上有两类工作队列: 普通工作队列和拷贝工作队列

+ 拷贝工作队列 : 优先队列
+ 普通工作队列 : FIFO

其中只有`cpu_priority_workder_`会在构造阶段被设置.

`cpu_normal_workers_`: 每个设备一个队列, 每个队列有`cpu_worker_nthreads_`个消费者(线程池大小).
`cpu_priority_worker_`: 只有唯一个队列, 有`cpu_priority_nthreads_`个消费者(线程池大小).
`gpu_normal_workers_`: 每个设备一个队列, 每个队列有`gpu_worker_nthreads_`个消费者(线程池大小).
`gpu_copy_workers_`: 每个设备一个队列, 每个队列有`gpu_worker_nthreads_`个消费者(线程池大小).

### 构造函数

+ 设置 `gpu_worker_nthreads_`, `cpu_worker_nthreads_`, `cpu_priority_nthreads_`, 
+ 设置 `cpu_priority_workers`: ThreadWorkerBlock<kPriorityQueue>
  - pool: `ThreadPool(cpu_priority_n_threads, CPUWorker)`

### override

`void PushToExecute(OprBlock *opr_block, bool pusher_thread);`
+ Exec Async and DeleteVar operation immediately if pushed from Pusher.
+ if is cpu task
  - enqueue the opr_block to `cpu_normal_worker` if is CPUPrioritized task. else
  - get `cpu_normal_worker`, push queue at front if is DeleteVar task, otherwise enqueue.
+ if is gpu task
  - if is copy task, get `gpu_copy_worker`
  - otherwise, get `gpu_normal_worker`


### 内部实现

```
//在特定设备上的 GPUWorker
template<dmlc::ConcurrentQueueType type>
inline void GPUWorker(Context ctx, //设备环境
                      bool is_copy_worker, // 是否是 copy 工作
                      ThreadWorkerBlock<type> *block, // the task block
                      std::shared_ptr<ThreadPool::SimpleEvent> ready_event); // 在设置完设备信息之后被激活.
```

```
//CPU worker that performs operations on CPU.
template<dmlc::ConcurrentQueueType type>
inline void CPUWorker(Context ctx,
                      ThreadWorkerBlock<type> *block); //The task block of the worker.
```