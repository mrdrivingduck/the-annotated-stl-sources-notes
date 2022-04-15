# Chapter 5.2 - RB-Tree

Created by : Mr Dk.

2021 / 04 / 06 16:32

Nanjing, Jiangsu, China

---

## RB-Tree Node

结点定义如下：

```c++
struct _Rb_tree_node_base
{
  typedef _Rb_tree_Color_type _Color_type;
  typedef _Rb_tree_node_base* _Base_ptr;

  _Color_type _M_color;  // 结点颜色
  _Base_ptr _M_parent;   // 父结点
  _Base_ptr _M_left;     // 左结点
  _Base_ptr _M_right;    // 右结点

  static _Base_ptr _S_minimum(_Base_ptr __x)
  {
    while (__x->_M_left != 0) __x = __x->_M_left;
    return __x;
  }

  static _Base_ptr _S_maximum(_Base_ptr __x)
  {
    while (__x->_M_right != 0) __x = __x->_M_right;
    return __x;
  }
};

template <class _Value>
struct _Rb_tree_node : public _Rb_tree_node_base
{
  typedef _Rb_tree_node<_Value>* _Link_type;
  _Value _M_value_field; // 红黑树结点
};
```

使用红黑树时，需要提供一个 `KeyOfValue` 仿函数，使得可以从 value 中获取到 key。对于 key 和 value 是同一个对象的情况 (比如说 set)，这样的仿函数被定义为：

```c++
template <class _Tp>
struct _Identity : public unary_function<_Tp,_Tp> {
  const _Tp& operator()(const _Tp& __x) const { return __x; } // key 的值就是 value 的值
};
```

## Insert

红黑树的两个重要的插入函数：

- `insert_unique()` - 不允许插入重复的结点，用于支持 set / map，返回值为 `pair<iterator, bool>`，`iterator` 指向插入位置，`bool` 表示是否插入成功
- `insert_equal()` - 允许插入重复的结点，用于支持 multiset / multimap，返回值为 `iterator`，指向插入位置

```c++
template <class _Key, class _Value, class _KeyOfValue,
          class _Compare, class _Alloc>
pair<typename _Rb_tree<_Key,_Value,_KeyOfValue,_Compare,_Alloc>::iterator,
     bool>
_Rb_tree<_Key,_Value,_KeyOfValue,_Compare,_Alloc>
  ::insert_unique(const _Value& __v)
{
  _Link_type __y = _M_header;
  _Link_type __x = _M_root(); // 根结点
  bool __comp = true;
  while (__x != 0) {
    __y = __x;
    __comp = _M_key_compare(_KeyOfValue()(__v), _S_key(__x)); // 待插入的 key 与当前结点比较 (通过 value 计算 key)
    __x = __comp ? _S_left(__x) : _S_right(__x); // 进入左子树或右子树
  }
  iterator __j = iterator(__y); // 插入点的父结点
  if (__comp) // 待插入 key < 插入点 key，插入在左侧
    if (__j == begin()) // 插入点父结点是最左结点
      return pair<iterator,bool>(_M_insert(__x, __y, __v), true); // (成功) 插入
    else
      --__j;
  if (_M_key_compare(_S_key(__j._M_node), _KeyOfValue()(__v))) // 待插入 key 与已有 key 不重复
    return pair<iterator,bool>(_M_insert(__x, __y, __v), true); // (成功) 插入
  return pair<iterator,bool>(__j, false); // key 重复，插入失败
}
```

```c++
template <class _Key, class _Value, class _KeyOfValue,
          class _Compare, class _Alloc>
typename _Rb_tree<_Key,_Value,_KeyOfValue,_Compare,_Alloc>::iterator
_Rb_tree<_Key,_Value,_KeyOfValue,_Compare,_Alloc>
  ::insert_equal(const _Value& __v)
{
  _Link_type __y = _M_header;
  _Link_type __x = _M_root(); // 根结点
  while (__x != 0) {
    __y = __x;
    __x = _M_key_compare(_KeyOfValue()(__v), _S_key(__x)) ?
            _S_left(__x) : _S_right(__x); // 根据 value 计算 key，进入左子树或右子树
  }
  return _M_insert(__x, __y, __v); // 插入 (肯定成功，除非结点内存分配失败)
}
```

## Find

从根结点开始搜寻是否存在某个特定的 key：

```c++
template <class _Key, class _Value, class _KeyOfValue,
          class _Compare, class _Alloc>
typename _Rb_tree<_Key,_Value,_KeyOfValue,_Compare,_Alloc>::const_iterator
_Rb_tree<_Key,_Value,_KeyOfValue,_Compare,_Alloc>::find(const _Key& __k) const
{
  _Link_type __y = _M_header; /* Last node which is not less than __k. */
  _Link_type __x = _M_root(); /* Current node. */

  while (__x != 0) {
    if (!_M_key_compare(_S_key(__x), __k))
      __y = __x, __x = _S_left(__x);  // key 小于当前结点 key，进入左子树
    else
      __x = _S_right(__x);            // key 大于当前结点 key，进入右子树
  }
  const_iterator __j = const_iterator(__y);  // 最终查找位置
  return (__j == end() || _M_key_compare(__k, _S_key(__j._M_node))) ? // 没找到
    end() : __j; // 找到了
}
```
