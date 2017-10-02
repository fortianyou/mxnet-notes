Executor
======

cpp-package是MxNet的C++前端.

include/mxnet-cpp/executor.h中定义封装了Executor 接口.


Forward
----

```
/*!
  * \brief Perform a Forward operation of Operator
  *  After this operation, user can get the result by using function head.
  */
  void Forward(bool is_train) {
    MXExecutorForward(handle_, is_train ? 1 : 0);
    mx_uint out_size;
    NDArrayHandle *out_array;
    CHECK_EQ(MXExecutorOutputs(handle_, &out_size, &out_array), 0);
    for (mx_uint i = 0; i < out_size; ++i) {
      outputs[i] = NDArray(out_array[i]);
    }
  }
```

Backward
----

```
/*!
  * \brief Perform a Backward operation of the Operator.
  *  This must be called after Forward.
  *  After this operation, NDArrays specified by grad_in_args_store will be
  *updated accordingly.
  *  User is allowed to pass in an empty Array if the head node is
  *  loss function and head gradeitn is not needed.
  *
  * \param head_grads the gradient of head nodes to be backproped.
  */
  void Backward(const std::vector<NDArray> &head_grads =
                    std::vector<NDArray>()) {
    std::vector<NDArrayHandle> head_grads_;
    for (auto d : head_grads) {
      head_grads_.push_back(d.GetHandle());
    }
    if (head_grads_.size() > 0) {
      MXExecutorBackward(handle_, head_grads_.size(), head_grads_.data());
    } else {
      MXExecutorBackward(handle_, 0, nullptr);
    }
  }
  ```


构造函数
-----

```
inline Executor::Executor(const Symbol &symbol, Context context,
                          const std::vector<NDArray> &arg_arrays,
                          const std::vector<NDArray> &grad_arrays,
                          const std::vector<OpReqType> &grad_reqs,
                          const std::vector<NDArray> &aux_arrays,
                          const std::map<std::string, Context> &group_to_ctx,
                          Executor *shared_exec) {
  this->arg_arrays = arg_arrays;
  this->grad_arrays = grad_arrays;
  this->aux_arrays = aux_arrays;
  this->symbol_ = symbol;

  std::vector<NDArrayHandle> arg_handles;
  std::vector<NDArrayHandle> grad_handles;
  std::vector<NDArrayHandle> aux_handles;

  for (const auto &array : arg_arrays) {
    arg_handles.push_back(array.GetHandle());
  }
  for (const auto &array : grad_arrays) {
    grad_handles.push_back(array.GetHandle());
  }
  for (const auto &array : aux_arrays) {
    aux_handles.push_back(array.GetHandle());
  }

  std::vector<mx_uint> grad_reqs_uint;
  for (auto s : grad_reqs) grad_reqs_uint.push_back(s);

  std::vector<const char *> map_keys;
  std::vector<int> dev_types, dev_ids;
  for (const auto &s : group_to_ctx) {
    map_keys.push_back(s.first.c_str());
    dev_types.push_back(s.second.GetDeviceType());
    dev_ids.push_back(s.second.GetDeviceId());
  }

  ExecutorHandle *shared_exec_handle =
      shared_exec == nullptr ? nullptr : &shared_exec->handle_;

  CHECK_EQ(MXExecutorBindEX(symbol.GetHandle(), context.GetDeviceType(),
                            context.GetDeviceId(), group_to_ctx.size(),
                            map_keys.data(), dev_types.data(), dev_ids.data(),
                            arg_handles.size(), arg_handles.data(),
                            grad_handles.data(), grad_reqs_uint.data(),
                            aux_handles.size(), aux_handles.data(),
                            shared_exec_handle, &handle_),
           0);

  mx_uint out_size;
  NDArrayHandle *out_array;
  CHECK_EQ(MXExecutorOutputs(handle_, &out_size, &out_array), 0);
  for (mx_uint i = 0; i < out_size; ++i) {
    outputs.push_back(NDArray(out_array[i]));
  }
}
```