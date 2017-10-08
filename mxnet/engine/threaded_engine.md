ThreadedEngine
=====

该类是所有 ThreadedEngine 的基类, 与继承类一起实现了生产者消费者模式.
在该类中, 主要实现了生产者消费者中生产者功能.

### OprBlock
每个 OprBlock 对应一个 push 到 ThreadedEngine 的 operation.
+ wait: 表示等待(pending task)计数
+ opr: ThreadOpr*, 表示实际的操作.
+ priority: 优先级

### VersionedVarBlock

对应 ThreadedVar 的一个状态版本, 是 ThreadedVar 的链表元素. 包含下一个版本的描述和触发操作.

+ next: VersionedVarBlock*
+ trigger: OprBlock *, 当前版本需要触发的操作
+ write: trigger 的操作是否会修改变量.

### ThreadedVar

在 ThreadEngine 中对应一个变量符号(用于描述依赖关系), 表示该变量上的动态操作, 内部实际上维护着一个 LinkedList 实现.

队列维护这因为写操作而造成等待的 OprBlock , pending list

+ head_: VersionedVarBlock*, 队尾元素
+ pending_write: VersionedVarBlock*, 队首元素
+ num_pending_reads: 正在读的 OprBlock 数.
+ is_ready_to_read: 当队列为空时返回 true, 表示变量可以被读.

### ThreadedOpr

Operator used in ThreadedEngine.

+ fn: AsyncFn
+ const_vars: vector<ThreadedVar*>
+ mutable_vars: vector<ThreadedVar*>
+ prop: FnProperty
+ opr_name: const char *
+ tempory: 是否是临时operator, 临时的一旦执行完毕便会删除.
