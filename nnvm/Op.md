Op
===
: brief Operator structure.

除了Op类内部定义的一些数据结构;
**Op采用注册机制**, 可以向Op注册关联一些额外的特性.

+ set_attr: 注册额外的属性
    + attr_name: 属性名, 每个属性名会对应一个[`OpMap`](#opmap)
    + value: 属性对象(`ValueType`) 
    + plevel: 优先级，默认为10, 优先级高的属性对象会替换优先级低的属性.

+ include
   注册某属性组中的所有属性对象.

<a id = 'opmap'>OpMap</a>
----
`OpMap<ValueType>`是一个关于`ValueType`类型对象的注册表.
一个注册了`ValueType`对象的`Op`会被记录到该表中(至多被注册一次).

+ 通过`Op`本身(`index_`)可以查询到所注册的`ValueType`对象.
+ 每个类型的注册表`OpMap<ValueType>`均是`OpManager::Global()->attrs`表中的一条记录.
+ 每个`OpMap<ValueType>`对象都会有一个唯一的字符串`attr_name_`与其对应.
+ 通过`Op::GetAttrMap(const std::string& key)`方法可以获得到全局属性表中的对应`OpMap<ValueType>`对象. 

### OpMap数据结构
```
template<typename ValueType>
class OpMap {
 public:
  inline const ValueType& operator[](const Op* op) const;
  inline const ValueType& get(const Op* op, const ValueType& def_value) const;
  inline int count(const Op* op) const;

 private:
  std::string attr_name_;
  std::vector<std::pair<ValueType, int> > data_;
  OpMap() = default;
};
```



Op数据结构片段
----

```
/* 
 * Besides the fields in the structure,
 * arbitary additional information can be associated with each op.
 * See function GetAttr for details.
 */
class Op {
 public:
  std::string name;
  std::string description;
  std::vector<ParamFieldInfo> arguments;
  uint32_t num_inputs = 1;

  std::function<uint32_t(const NodeAttrs& attrs)> get_num_outputs = nullptr;
  std::function<uint32_t(const NodeAttrs& attrs)> get_num_inputs = nullptr;
  std::function<void(NodeAttrs* attrs)> attr_parser = nullptr;

  static const Op* Get(const std::string& op_name);
  template<typename ValueType>
  static const OpMap<ValueType>& GetAttr(const std::string& attr_name);

 private:
  uint32_t index_{0};
  static const any* GetAttrMap(const std::string& key);
  static void UpdateAttrMap(const std::string& key,
                            std::function<void(any*)> updater);
  static void AddGroupTrigger(const std::string& group_name,
                              std::function<void(Op*)> trigger);

  /*!
   * \brief Register additional attributes to operator.
   * \param attr_name The name of the attribute.
   * \param value The value to be set.
   * \param plevel The priority level of this set,
   *  an higher priority level attribute
   *  will replace lower priority level attribute.
   *  Must be bigger than 0.
   *
   *  Cannot set with same plevel twice in the code.
   *
   * \tparam ValueType The type of the value to be set.
   */
  template<typename ValueType>
  inline Op& set_attr(const std::string& attr_name,  // NOLINT(*)
                      const ValueType& value,
                      int plevel = 10);

};
```
