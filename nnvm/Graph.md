Graph
====


<a id="graph">`Graph`</a>
----
Graph本身图连接结构蕴含在outputs的依赖关系中.
```
class Graph {
  std::vector<NodeEntry> outputs; //outputs of the computation graph.
  std::unordered_map<std::string, std::shared_ptr<any> > attrs; //attributes of a graph
  std::shared_ptr<const IndexedGraph> indexed_graph_;
};
```

`IndexedGraph`
-----
: 通过DFS遍历[`Graph`](#graph).outputs构建.
+ `entry_rptr_`: CSR pointer of node entries
  
  `entry_rptr_.push_back(entry_rptr_.back() + node.num_outputs()) ` 
+ `input_entrys_`: 保存所有[`Node`](#node)的所有输入项, 一个项的唯一标志是`(node_id, index, version)`
+ `output_entrys_`: 保存所有[`Node`](#node)的所有输出项, 一个项的唯一标志是`(node_id, index, version)`

```
class IndexedGraph {
 public:
  /*! represents a data in the graph */
  struct NodeEntry {
    uint32_t node_id; //the source node id in the computation graph 
    uint32_t index; //index of output from the source. 
  };
  /*! Node data structure in IndexedGraph */
  struct Node {
    /*! pointer to the source node */
    const nnvm::Node* source;
    array_view<NodeEntry> inputs;
    array_view<uint32_t> control_deps;
  };

  std::vector<Node> nodes_;
  std::vector<uint32_t> input_nodes_;  // Index to all input nodes.
  std::unordered_set<uint32_t> mutable_input_nodes_;  // Index to all mutable input nodes.
  std::vector<NodeEntry> outputs_;  // space to store the outputs entries
  std::unordered_map<const nnvm::Node*, uint32_t> node2index_;  // mapping from node to index.
  std::vector<size_t> entry_rptr_;  // CSR(Compressed Sparse Row) pointer of node entries
  std::vector<NodeEntry> input_entries_;  // space to store input entries of each
  std::vector<uint32_t> control_deps_;  // control flow dependencies
};
```

<a id='node'>`Node`</a>
----
```
class Node{
  NodeAttrs attrs; //定义在Op等属性
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

```
struct NodeAttrs {
  const Op *op{nullptr};
  std::string name;
  std::vector<double> scalars;
  std::unordered_map<std::string, std::string> dict;
  any parsed;
};
```