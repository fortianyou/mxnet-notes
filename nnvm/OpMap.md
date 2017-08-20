OpMap 
===
A map data structure that takes `Op*` as key and returns ValueType
```

/*!
 * \tparam ValueType The type of the value stored in map.
 */
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

OpManager::Global()->attr: 表示全局的OpMap
```

// Get attribute map by key
const any* Op::GetAttrMap(const std::string& key) {
  auto& dict =  OpManager::Global()->attr;
  auto it = dict.find(key);
  if (it != dict.end()) {
    return it->second.get();
  } else {
    return nullptr;
  }
}
```

OpMap<ValueType>的元数据是个数组, 数组的每个元素对应一个Op实例. 所以count返回0或1.
```
// member functions of OpMap
template<typename ValueType>
inline int OpMap<ValueType>::count(const Op* op) const {
  if (op == nullptr) return 0;
  const uint32_t idx = op->index_;
  return idx < data_.size() ? (data_[idx].second != 0) : 0;
}
```