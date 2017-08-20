
<a id='node'>`Node`</a>
----
`Node`表示Graph中的一个操作. 所有Operation相关的属性通过[`NodeAttrs`](#attrs)结构体来定义.

```
class Node{
  NodeAttrs attrs; //the attributes in the node
  std::vector<NodeEntry> inputs; 
  std::vector<NodePtr> control_deps; //Optional control flow dependencies

  //判断Node是否是一个placeholder变量
  inline bool is_variable() const;
};
```

NodeEntry
----
: an entry that represents output data from a [`Node`](#node)
```
struct NodeEntry {
  share_ptr<Node> node;
  uint32_t index;
};
```

<a id='attrs'>`NodeAttrs`</a>
-----
: The Attributes of the current operation [`Node`](#node).

Attributes包括如下内容：
+ 具体的Op实例 (操作)
+ 如果[`Node`](#node)是一个变量，则`op == nullptr`
+ 可以指向any对象的parsed变量. 该变量是`NodeAttrs`的重要属性.

如果是变量
----
+ `op == nullptr`
+ `num_inputs() == 1`

```
struct NodeAttrs {
  const Op *op{nullptr};
  std::string name;
  std::vector<double> scalars;
  std::unordered_map<std::string, std::string> dict;
  any parsed;
};
```

回答问题NodeAttrs的作用，以及如何被构建


回答问题control_deps如何起作用？