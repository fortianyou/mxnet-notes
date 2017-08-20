Graph
====


<a id="graph">`Graph`</a>
----
Graph表示符号计算图, Graph本身图连接结构蕴含在outputs的依赖关系中. 用于构建IndexedGraph
```
class Graph {
  std::vector<NodeEntry> outputs; //outputs of the computation graph.
  std::unordered_map<std::string, std::shared_ptr<any> > attrs; //attributes of a graph
  std::shared_ptr<const IndexedGraph> indexed_graph_;
};
```

`IndexedGraph`
-----
```
Auxiliary data structure to index a graph.
It maps Nodes in the graph to consecutive integers node_id.
It also maps IndexedGraph::NodeEntry to consecutive integer entry_id.
This allows storing properties of Node and NodeEntry into
compact vector and quickly access them without resorting to hashmap.
 
The node_id and entry_rptr are the same as the JSON graph produced by SaveJSON Pass.
```
: 通过DFS遍历[`Graph`](#graph).outputs构建.

### Indexed构建过程：
+ DFS遍历`Graph.outputs`: [`Node`](#node)，设置如下字段
   + 添加`[Node](#node)`到`IndexedGraph.nodes_`, 并设置`IndexedGraph.node2index_`
   + 如果`[Node](#node).is_variable() == true`, 则添加到`IndexedGraph.input_nodes_`表示全局静态数据输入节点.
   + 设置`IndexedGraph.entry_rptr_`, `entry_rptr_[i]`对应`nodes_[i]`的output entry的起始位置.
   + 遍历当前`Node->inputs`的设置`IndexedGraph.input_entries_`(表示整张图的所有数据输入项), 每个元素都是`IndexedGraph.NodeEntry`.
   + 设置`IndexedGraph.control_deps_`

+ 添加`Graph.outputs`: [`Node`](#node)到`IndexedGraph.outputs_: IndexGraph.NodeEntry`表示整张图的所有数据输出项.

+ 设置`IndexedGraph.nodes_[i]`的相关信息
   + 按照`IndexedGraph.input_entries_`设置`nodes_[i].inputs`和`IndexedGraph.mutable_input_nodes_`
   + 按照`control_deps_`设置`nodes_[i].control_deps`


+ `input_entrys_`: 保存`IndexedGraph`所有[`Node`](#node)的所有输入项, 一个项的唯一标志是`(node_id, index, version)`
+ `outputs_`: 保存`IndexedGraph`所有[`Node`](#node)的所有输出项, 一个项的唯一标志是`(node_id, index, version)`
+ `control_deps_`: 保存`IndexedGraph`所有的控制依赖关系.

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
  NodeAttrs attrs; // the attributes in the node.
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

