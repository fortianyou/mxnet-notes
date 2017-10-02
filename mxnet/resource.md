Resources
====

Manages global resources, such as the random number generator and temporal space.


SpaceAllocator
-----

```
// internal structure for space allocator
struct SpaceAllocator {
  // internal context
  Context ctx;
  // internal handle
  Storage::Handle handle;
  // internal CPU handle
  Storage::Handle host_handle;
};
```

