# Chapter 5.11 - hash_multimap

Created by : Mr Dk.

2021 / 04 / 06 23:43

Nanjing, Jiangsu, China

---

SGI STL 的 hash_multimap 以 hashtable 作为底层机制。

- 与 multimap (底层为 RB-Tree) 的区别：无序性
- 与 hash_map 的区别：允许 key 值重复

```c++
template <class _Key, class _Tp,
          class _HashFcn  __STL_DEPENDENT_DEFAULT_TMPL(hash<_Key>),
          class _EqualKey __STL_DEPENDENT_DEFAULT_TMPL(equal_to<_Key>),
          class _Alloc =  __STL_DEFAULT_ALLOCATOR(_Tp) >
class hash_multimap;
```

与 hash_map 使用的底层结构类似，因此代码基本相同。比较明显的区别是，构造函数或插入函数中，使用的是 `insert_equal()` 而不是 `insert_unique()`：

```c++
iterator insert(const value_type& __obj)
    { return _M_ht.insert_equal(__obj); }
void insert(const value_type* __f, const value_type* __l) {
    _M_ht.insert_equal(__f,__l);
}
void insert(const_iterator __f, const_iterator __l)
    { _M_ht.insert_equal(__f, __l); }
```

另外，以下三个函数将能够返回任意值或区间 (这是 key 重复的特性决定的)：

```c++
size_type count(const key_type& __key) const { return _M_ht.count(__key); }

pair<iterator, iterator> equal_range(const key_type& __key)
  { return _M_ht.equal_range(__key); }
pair<const_iterator, const_iterator>
    equal_range(const key_type& __key) const
  { return _M_ht.equal_range(__key); }
```
