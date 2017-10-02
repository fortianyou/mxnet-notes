Symbol
===
cpp-package是MxNet的C++前端.

include/mxnet-cpp/symbol.h中定义封装了Symbol接口.



SymBlob
----
用于存储`SymbolHandler`的结构体.


Symbol 类型及创建方法
----
+ Variable: `Symbol::Variable(std::string name)`
+ Group: `Symbol::Group(std::vector<Symbol> list)`
+ Load: `Symbol::Load(std::string filename)`
+ Json: `Symbol::LoadJSON(std::string json_str)`


```
  void InferShape(
      const map<string, vector<mx_uint> > &arg_shapes,
      vector<vector<mx_uint> > *in_shape,
      vector<vector<mx_uint> > *aux_shape,
      vector<vector<mx_uint> > *out_shape) const;

  void InferExecutorArrays(
      const Context &context,
      vector<NDArray> *arg_arrays,
      vector<NDArray> *grad_arrays, 
      vector<OpReqType> *grad_reqs,
      vector<NDArray> *aux_arrays,
      const map<string, NDArray> &args_map,
      const map<string, NDArray> &arg_grad_store = map<string, NDArray>(),
      const map<string, OpReqType> &grad_req_type = map<string, OpReqType>(),
      const map<string, NDArray> &aux_map = map<string, NDArray>()) const;

  void InferArgsMap(const Context &context,
                    map<string, NDArray> *args_map,
                    const map<string, NDArray> &known_args) const;
 /*!
  *  Create an executor by bind symbol with context and arguments.
  *  If user do not want to compute the gradients of i-th argument,
  *  grad_req_type[i] can be kNullOp.
  *  The input arrays in the given maps should have the same name with the input symbol.
  *  Only need some of the necessary arrays, and the other arrays can be infered automatically.
  */ 
  Executor *SimpleBind(const Context &context,
                       const map<string, NDArray> &args_map,
                       const map<string, NDArray> &arg_grad_store = map<string, NDArray>(),
                       const map<string, OpReqType> &grad_req_type = map<string, OpReqType>(),
                       const map<string, NDArray> &aux_map = map<string, NDArray>());

 /*!
  * Create an executor by bind symbol with context and arguments.
  * If user do not want to compute the gradients of i-th argument,
  * grad_req_type[i] can be kNullOp.
  */
  Executor *Bind(const Context &context, 
                 const vector<NDArray> &arg_arrays,
                 const vector<NDArray> &grad_arrays,
                 const vector<OpReqType> &grad_reqs,
                 const vector<NDArray> &aux_arrays,
                 const map<string, Context> &group_to_ctx = map<string, Context>(),
                 Executor *shared_exec = nullptr);


```