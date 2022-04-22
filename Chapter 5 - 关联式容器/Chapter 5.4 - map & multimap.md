# Chapter 5.3 - map & multimap

Created by : Mr Dk.

2021 / 04 / 06 17:22

Nanjing, Jiangsu, China

---

map 的特性是，所有元素都根据元素的 key 值自动被排序。map 的所有元素都是 **pair** (key + value)。map 不允许两个元素有相同的 key 值。

```cpp
template <class _Key, class _Tp,
          class _Compare __STL_DEPENDENT_DEFAULT_TMPL(less<_Key>), // 默认使用 < 比较 key
          class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) > // 默认使用 alloc 分配器
class map;
```

multimap 允许两个元素有相同的 key。

```cpp
template <class _Key, class _Tp,
          class _Compare __STL_DEPENDENT_DEFAULT_TMPL(less<_Key>),
          class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
class multimap;
```

## Pair

```cpp
template <class _T1, class _T2>
struct pair {
  typedef _T1 first_type;
  typedef _T2 second_type;

  _T1 first;  // key
  _T2 second; // value
  pair() : first(_T1()), second(_T2()) {}
  pair(const _T1& __a, const _T2& __b) : first(__a), second(__b) {}
};
```

## Definition

```cpp
template <class _Key, class _Tp, class _Compare, class _Alloc>
class map {
public:

// requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_BINARY_FUNCTION_CHECK(_Compare, bool, _Key, _Key);

// typedefs:

  // 类内类型定义
  typedef _Key                  key_type;
  typedef _Tp                   data_type;
  typedef _Tp                   mapped_type;
  typedef pair<const _Key, _Tp> value_type;
  typedef _Compare              key_compare;

  // 定义用于比较 value 相对大小的仿函数，重载了该类的 () 运算符
  // 使用与泛型参数 (用于比较 key 的仿函数) 相同的仿函数比较 value
  class value_compare
    : public binary_function<value_type, value_type, bool> {
  friend class map<_Key,_Tp,_Compare,_Alloc>;
  protected :
    _Compare comp;
    value_compare(_Compare __c) : comp(__c) {}
  public:
    bool operator()(const value_type& __x, const value_type& __y) const {
      return comp(__x.first, __y.first);
    }
  };

private:
  typedef _Rb_tree<key_type, value_type, // 红黑树的类型定义
                   _Select1st<value_type>, key_compare, _Alloc> _Rep_type;
  _Rep_type _M_t;  // red-black tree representing map

  // ...
};
```

其中，红黑树的定义里，key 的类型是来自于模板参数，value 的类型为模板参数 key 和 value 组成的 pair。key 比较函数来自于模板参数。`KeyOfValue` 仿函数的功能是从红黑树的 value 类型 pair 中取出 pair 的 key：

```cpp
template <class _Pair>
struct _Select1st : public unary_function<_Pair, typename _Pair::first_type> {
  const typename _Pair::first_type& operator()(const _Pair& __x) const {
    return __x.first;
  }
};
```

## Compare

类内封装了一个用于比较 key 的仿函数，重载了该类的 `()` 运算符：

```cpp
class value_compare : public binary_function<value_type, value_type, bool> {
    friend class map<_Key,_Tp,_Compare,_Alloc>;
protected :
    _Compare comp;
    value_compare(_Compare __c) : comp(__c) {}
public:
    bool operator()(const value_type& __x, const value_type& __y) const {
        return comp(__x.first, __y.first);
    }
};
```

## Constructor

构造函数初始化红黑树：

```cpp
map() : _M_t(_Compare(), allocator_type()) {}
explicit map(const _Compare& __comp,
             const allocator_type& __a = allocator_type())
    : _M_t(__comp, __a) {}

map(const value_type* __first, const value_type* __last)
    : _M_t(_Compare(), allocator_type())
    { _M_t.insert_unique(__first, __last); }

map(const value_type* __first,
    const value_type* __last, const _Compare& __comp,
    const allocator_type& __a = allocator_type())
    : _M_t(__comp, __a) { _M_t.insert_unique(__first, __last); }

map(const_iterator __first, const_iterator __last)
    : _M_t(_Compare(), allocator_type())
    { _M_t.insert_unique(__first, __last); }

map(const_iterator __first, const_iterator __last, const _Compare& __comp,
    const allocator_type& __a = allocator_type())
    : _M_t(__comp, __a) { _M_t.insert_unique(__first, __last); }
```

## Accessors

直接转而调用红黑树的函数：

```cpp
key_compare key_comp() const { return _M_t.key_comp(); }
value_compare value_comp() const { return value_compare(_M_t.key_comp()); }
allocator_type get_allocator() const { return _M_t.get_allocator(); }

iterator begin() { return _M_t.begin(); }
const_iterator begin() const { return _M_t.begin(); }
iterator end() { return _M_t.end(); }
const_iterator end() const { return _M_t.end(); }
reverse_iterator rbegin() { return _M_t.rbegin(); }
const_reverse_iterator rbegin() const { return _M_t.rbegin(); }
reverse_iterator rend() { return _M_t.rend(); }
const_reverse_iterator rend() const { return _M_t.rend(); }
bool empty() const { return _M_t.empty(); }
size_type size() const { return _M_t.size(); }
size_type max_size() const { return _M_t.max_size(); }
_Tp& operator[](const key_type& __k) { // 重载 [] 运算符
    iterator __i = lower_bound(__k); // 第一个大于等于指定 key 的结点
    // __i->first is greater than or equivalent to __k.
    if (__i == end() || key_comp()(__k, (*__i).first))
        __i = insert(__i, value_type(__k, _Tp())); // key 不存在，则插入新结点
    return (*__i).second; // 返回 key 结点对应 value 的左值引用：map[key] = value;
}
void swap(map<_Key,_Tp,_Compare,_Alloc>& __x) { _M_t.swap(__x._M_t); }
```

## Insert

不允许元素的 key 重复，因此调用红黑树的 `insert_unique()`；如果是 multimap，那么调用 `insert_equal()`。

```cpp
pair<iterator,bool> insert(const value_type& __x)
  { return _M_t.insert_unique(__x); }
iterator insert(iterator position, const value_type& __x)
  { return _M_t.insert_unique(position, __x); }

void insert(const value_type* __first, const value_type* __last) {
    _M_t.insert_unique(__first, __last);
}
void insert(const_iterator __first, const_iterator __last) {
    _M_t.insert_unique(__first, __last);
}
```

## Erase

```cpp
void erase(iterator __position) { _M_t.erase(__position); }
size_type erase(const key_type& __x) { return _M_t.erase(__x); }
void erase(iterator __first, iterator __last)
{ _M_t.erase(__first, __last); }
void clear() { _M_t.clear(); }
```

## Search

由于红黑树的有序性，因此可以使用二分查找：

```cpp
iterator find(const key_type& __x) { return _M_t.find(__x); }
const_iterator find(const key_type& __x) const { return _M_t.find(__x); }
size_type count(const key_type& __x) const {
    return _M_t.find(__x) == _M_t.end() ? 0 : 1;
}
iterator lower_bound(const key_type& __x) {return _M_t.lower_bound(__x); }
const_iterator lower_bound(const key_type& __x) const {
    return _M_t.lower_bound(__x);
}
iterator upper_bound(const key_type& __x) {return _M_t.upper_bound(__x); }
const_iterator upper_bound(const key_type& __x) const {
    return _M_t.upper_bound(__x);
}

pair<iterator,iterator> equal_range(const key_type& __x) {
    return _M_t.equal_range(__x);
}
pair<const_iterator,const_iterator> equal_range(const key_type& __x) const {
    return _M_t.equal_range(__x);
}
```

## Operator Overload

借用红黑树的运算符实现重载：

```cpp
template <class _Key, class _Tp, class _Compare, class _Alloc>
inline bool operator==(const map<_Key,_Tp,_Compare,_Alloc>& __x,
                       const map<_Key,_Tp,_Compare,_Alloc>& __y) {
  return __x._M_t == __y._M_t;
}

template <class _Key, class _Tp, class _Compare, class _Alloc>
inline bool operator<(const map<_Key,_Tp,_Compare,_Alloc>& __x,
                      const map<_Key,_Tp,_Compare,_Alloc>& __y) {
  return __x._M_t < __y._M_t;
}
```
