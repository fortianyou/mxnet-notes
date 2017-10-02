Pooled Storage
====

```
/*!
 * \brief Storage manager with a memory pool on gpu.
 */
class GPUPooledStorageManager final : public StorageManager {
 public:
  /*!
   * \brief Default constructor.
   */
  GPUPooledStorageManager() {
    reserve_ = dmlc::GetEnv("MXNET_GPU_MEM_POOL_RESERVE", 5);
  }
 private:
  void ReleaseAll();
  // internal mutex
  std::mutex mutex_;
  // used memory
  size_t used_memory_ = 0;
  // percentage of reserved memory
  int reserve_;
  // number of devices
  const int NDEV = 32;
  // memory pool
  std::unordered_map<size_t, std::vector<void*>> memory_pool_;
  DISALLOW_COPY_AND_ASSIGN(GPUPooledStorageManager);
};  // class GPUPooledStorageManager
```


Alloc
----

Alloc 的代码如下:

```
void* GPUPooledStorageManager::Alloc(size_t raw_size) {
  std::lock_guard<std::mutex> lock(mutex_);
  size_t size = raw_size + NDEV;
  auto&& reuse_it = memory_pool_.find(size);
  if (reuse_it == memory_pool_.end() || reuse_it->second.size() == 0) {
    size_t free, total;
    cudaMemGetInfo(&free, &total);
    if (free <= total * reserve_ / 100 || size > free - total * reserve_ / 100)
      ReleaseAll();

    void* ret = nullptr;
    cudaError_t e = cudaMalloc(&ret, size);
    if (e != cudaSuccess && e != cudaErrorCudartUnloading) {
      LOG(FATAL) << "cudaMalloc failed: " << cudaGetErrorString(e);
    }
    used_memory_ += size;
    return ret;
  } else {
    auto&& reuse_pool = reuse_it->second;
    auto ret = reuse_pool.back();
    reuse_pool.pop_back();
    return ret;
  }
}


void GPUPooledStorageManager::ReleaseAll() {
  for (auto&& i : memory_pool_) {
    for (auto&& j : i.second) {
      DirectFree(j, i.first - NDEV);
    }
  }
  memory_pool_.clear();
}
```

size 表示一次申请所需要的字节数

从 memory_pool 查找对应长度的可用内存

+ 如果存在, 则直接复用.
+ 如果不存在, 则从显存中分配内存.
   - 如果显存不够, 则释放内存池中的所有内存再分配.
