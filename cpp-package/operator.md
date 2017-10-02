operator
====

cpp-package是MxNet的C++前端.

include/mxnet-cpp/operator.h中定义封装了Operator接口.


构造函数
---
Operator 的构造函数代码如下:

```
inline Operator::Operator(const std::string &operator_name) {
  handle_ = op_map()->GetSymbolCreator(operator_name);
  const char *name;
  const char *description;
  mx_uint num_args;
  const char **arg_names;
  const char **arg_type_infos;
  const char **arg_descriptions;
  const char *key_var_num_args;
  MXSymbolGetAtomicSymbolInfo(handle_,
      &name,
      &description,
      &num_args,
      &arg_names,
      &arg_type_infos,
      &arg_descriptions,
      &key_var_num_args);
  for (mx_uint i = 0; i < num_args; ++i) {
    arg_names_.push_back(arg_names[i]);
  }
}
```

 此构造函数将`op_map()->GetSymbolCreator(operator_name)`返回的句柄, 作为`MXSymbolGetAtomicSymbolInfo` 的参数, 查询`Operator`的参数信息.

参数与输入设置
----
Operator 接口主要包括 SetParam 和 SetInput 等方法.


创建符号`CreateSymbol`
----
创建符号调用了`MXSymbolCreateAtomicSymbol` 和 `MXSymbolCompose` 两个方法.
```
inline Symbol Operator::CreateSymbol(const std::string &name) {
  SymbolHandle symbol_handle;
  ...
  MXSymbolCreateAtomicSymbol(handle_, param_keys.size(), param_keys.data(),
                             param_values.data(), &symbol_handle);
  MXSymbolCompose(symbol_handle, pname, input_symbols_.size(), input_keys_p,
                  input_symbols_.data());
  return Symbol(symbol_handle);
}
```

`Invoke` 函数
----
```
inline void Operator::Invoke(std::vector<NDArray> &outputs) {
  if (input_keys_.size() > 0) {
    CHECK_EQ(input_keys_.size(), input_ndarrays_.size());
  }

  std::vector<const char *> input_keys;
  std::vector<const char *> param_keys;
  std::vector<const char *> param_values;

  for (auto &data : params_) {
    param_keys.push_back(data.first.c_str());
    param_values.push_back(data.second.c_str());
  }

  int num_inputs = input_ndarrays_.size();
  int num_outputs = outputs.size();
  std::vector<NDArrayHandle> output_handles;
  std::transform(outputs.begin(), outputs.end(),
      std::back_inserter(output_handles), [](NDArray& a) {
        return a.GetHandle();
      });

  NDArrayHandle *outputs_receiver = nullptr;
  if (num_outputs > 0) {
    outputs_receiver = output_handles.data();
  }

  MXImperativeInvoke(handle_, num_inputs, input_ndarrays_.data(),
      &num_outputs, &outputs_receiver,
      param_keys.size(), param_keys.data(), param_values.data());

  if (outputs.size() > 0)
    return;

  std::transform(outputs_receiver, outputs_receiver+num_outputs,
      std::back_inserter(outputs), [](const NDArrayHandle& handle) {
        return NDArray(handle);
      });
}
```
