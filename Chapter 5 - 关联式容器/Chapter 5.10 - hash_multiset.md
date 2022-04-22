# Chapter 5.10 - hash_multiset

Created by : Mr Dk.

2021 / 04 / 06 23:41

Nanjing, Jiangsu, China

---

SGI STL 的 hash_multiset 以 hashtable 作为底层机制。

- 与 multiset (底层为 RB-Tree) 的区别：无序性
- 与 hash_set 的区别：允许 key 值重复

```cpp
template <class _Value,
          class _HashFcn  __STL_DEPENDENT_DEFAULT_TMPL(hash<_Value>),
          class _EqualKey __STL_DEPENDENT_DEFAULT_TMPL(equal_to<_Value>),
          class _Alloc =  __STL_DEFAULT_ALLOCATOR(_Value) >
class hash_multiset;
```

与 hash_set 使用的底层结构类似，因此代码基本相同。比较明显的区别是，构造函数或插入函数中，使用的是 `insert_equal()` 而不是 `insert_unique()`：

```cpp
iterator insert(const value_type& __obj)
    { return _M_ht.insert_equal(__obj); }
void insert(const value_type* __f, const value_type* __l) {
    _M_ht.insert_equal(__f,__l);
}
void insert(const_iterator __f, const_iterator __l)
    { _M_ht.insert_equal(__f, __l); }
```

另外，以下两个函数将能够返回任意值或区间 (这是 key 重复的特性决定的)：

```cpp
size_type count(const key_type& __key) const { return _M_ht.count(__key); } // 返回指定 key 值的个数

pair<iterator, iterator> equal_range(const key_type& __key) const // 返回指定 key 值的迭代器区间
  { return _M_ht.equal_range(__key); }
```
