# Chapter 5.3 - set & multiset

Created by : Mr Dk.

2021 / 04 / 06 16:57

Nanjing, Jiangsu, China

---

set 的特性是，所有元素会根据元素的 key 自动排序。set 元素的 key 就是 value，并且不允许两个元素有相同的 key。另外，无法通过 set 的迭代器修改 set 的元素值：因为这将会破坏 set 的组织。

multiset 允许两个元素有相同的 key。

set 的迭代器在经历插入或删除操作后一般来说不会失效。

STL 特别提供了一组 set / multiset 的算法来进行集合操作：

* `set_intersection()` 交集
* `set_union()` 并集
* `set_difference()` 差集
* `set_symmetric_difference()` 对称差集

标准 STL 以 **红黑树** 作为 set 的底层机制。

```c++
template <class _Key, class _Compare __STL_DEPENDENT_DEFAULT_TMPL(less<_Key>), // 默认使用 < 比较 key
          class _Alloc = __STL_DEFAULT_ALLOCATOR(_Key) > // 默认使用 alloc 分配器
class set;
```

```c++
template <class _Key, class _Compare, class _Alloc>
class set {
  // requirements:

  __STL_CLASS_REQUIRES(_Key, _Assignable);
  __STL_CLASS_BINARY_FUNCTION_CHECK(_Compare, bool, _Key, _Key);

public:
  // typedefs:

  typedef _Key     key_type;
  typedef _Key     value_type; // key and value are the same
  typedef _Compare key_compare;
  typedef _Compare value_compare;
private:
  typedef _Rb_tree<key_type, value_type, 
                  _Identity<value_type> /* KeyOfValue */, key_compare, _Alloc> _Rep_type;
  _Rep_type _M_t;  // red-black tree representing set
  
public: // 只读迭代器，不可修改 key 的值
  typedef typename _Rep_type::const_pointer pointer;
  typedef typename _Rep_type::const_pointer const_pointer;
  typedef typename _Rep_type::const_reference reference;
  typedef typename _Rep_type::const_reference const_reference;
  typedef typename _Rep_type::const_iterator iterator;
  typedef typename _Rep_type::const_iterator const_iterator;
  typedef typename _Rep_type::const_reverse_iterator reverse_iterator;
  typedef typename _Rep_type::const_reverse_iterator const_reverse_iterator;
  typedef typename _Rep_type::size_type size_type;
  typedef typename _Rep_type::difference_type difference_type;
  typedef typename _Rep_type::allocator_type allocator_type;
    
  // ...
};
```

其中，红黑树的 `KeyOfValue` 仿函数被定义为 `identity`：key 即 value。

```c++
template <class _Tp>
struct _Identity : public unary_function<_Tp,_Tp> {
  const _Tp& operator()(const _Tp& __x /* value */ ) const { return __x /* key */; }
};
```

此外，基本上所有的 set 操作都转而调用红黑树的函数。

构造函数需要将 **元素比较仿函数** 传给红黑树：

```c++
set() : _M_t(_Compare(), allocator_type()) {}
explicit set(const _Compare& __comp,
             const allocator_type& __a = allocator_type())
    : _M_t(__comp, __a) {}

set(const value_type* __first, const value_type* __last) 
    : _M_t(_Compare(), allocator_type()) 
    { _M_t.insert_unique(__first, __last); }

set(const value_type* __first, 
    const value_type* __last, const _Compare& __comp,
    const allocator_type& __a = allocator_type())
    : _M_t(__comp, __a) { _M_t.insert_unique(__first, __last); }

set(const_iterator __first, const_iterator __last)
    : _M_t(_Compare(), allocator_type()) 
    { _M_t.insert_unique(__first, __last); }

set(const_iterator __first, const_iterator __last, const _Compare& __comp,
    const allocator_type& __a = allocator_type())
    : _M_t(__comp, __a) { _M_t.insert_unique(__first, __last); }
```

元素访问函数：

```c++
key_compare key_comp() const { return _M_t.key_comp(); }
value_compare value_comp() const { return _M_t.key_comp(); }
allocator_type get_allocator() const { return _M_t.get_allocator(); }

iterator begin() const { return _M_t.begin(); }
iterator end() const { return _M_t.end(); }
reverse_iterator rbegin() const { return _M_t.rbegin(); } 
reverse_iterator rend() const { return _M_t.rend(); }
bool empty() const { return _M_t.empty(); }
size_type size() const { return _M_t.size(); }
size_type max_size() const { return _M_t.max_size(); }
void swap(set<_Key,_Compare,_Alloc>& __x) { _M_t.swap(__x._M_t); }
```

插入函数。由于 set 不允许元素重复出现，因此在插入结点时，调用的是 `insert_unique()`。如果是 multiset，那么调用的是 `insert_equal()`。

```c++
pair<iterator,bool> insert(const value_type& __x) {           // (从根结点开始寻找位置) 并插入元素
    pair<typename _Rep_type::iterator, bool> __p = _M_t.insert_unique(__x); 
    return pair<iterator, bool>(__p.first, __p.second);
}
iterator insert(iterator __position, const value_type& __x) { // 在指定位置插入元素
    typedef typename _Rep_type::iterator _Rep_iterator;
    return _M_t.insert_unique((_Rep_iterator&)__position, __x);
}

void insert(const_iterator __first, const_iterator __last) {
    _M_t.insert_unique(__first, __last);
}
void insert(const value_type* __first, const value_type* __last) {
    _M_t.insert_unique(__first, __last);
}
```

删除函数：

```c++
void erase(iterator __position) { 
    typedef typename _Rep_type::iterator _Rep_iterator;
    _M_t.erase((_Rep_iterator&)__position); 
}
size_type erase(const key_type& __x) { 
    return _M_t.erase(__x); 
}
void erase(iterator __first, iterator __last) { 
    typedef typename _Rep_type::iterator _Rep_iterator;
    _M_t.erase((_Rep_iterator&)__first, (_Rep_iterator&)__last); 
}
void clear() { _M_t.clear(); }
```

查找函数。由于元素有序，因此可以进行二分范围查找：

```c++
iterator find(const key_type& __x) const { return _M_t.find(__x); } // 查找确切元素
size_type count(const key_type& __x) const { // 查找特定元素出现的个数 (只能为 0 或 1)
    return _M_t.find(__x) == _M_t.end() ? 0 : 1;
}
iterator lower_bound(const key_type& __x) const { // 第一个大于等于 x 的结点
    return _M_t.lower_bound(__x);
}
iterator upper_bound(const key_type& __x) const { // 第一个大于 x 的结点
    return _M_t.upper_bound(__x); 
}
pair<iterator,iterator> equal_range(const key_type& __x) const { // 与 x 值相同的元素范围
    return _M_t.equal_range(__x);
}
```

比较运算符的重载直接借用了底层红黑树的运算符：

```c++
template <class _Key, class _Compare, class _Alloc>
inline bool operator==(const set<_Key,_Compare,_Alloc>& __x, 
                       const set<_Key,_Compare,_Alloc>& __y) {
  return __x._M_t == __y._M_t;
}

template <class _Key, class _Compare, class _Alloc>
inline bool operator<(const set<_Key,_Compare,_Alloc>& __x, 
                      const set<_Key,_Compare,_Alloc>& __y) {
  return __x._M_t < __y._M_t;
}
```

---

