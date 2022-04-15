# Chapter 5.7 - hashtable

Created by : Mr Dk.

2021 / 04 / 06 23:24

Nanjing, Jiangsu, China

---

## 5.7.1 hashtable 概述

二叉搜索树具有对数平均时间的表现，前提是 **输入数据具有足够的随机性**。而哈希表在插入、删除、搜寻等操作上都具有常数平均时间的表现，并且以统计为基础，不需要依赖输入元素的随机性。

使用 hash 函数带来的问题是有不同的元素被映射到相同的位置上，即所谓 **碰撞** 问题。解决碰撞的方法：

- 线性探测 (linear probing)：负载因子小于 1，按照计算所得的 hash 值的位置逐一向下寻找 (平均插入成本的增长幅度远高于负载系数的增长幅度)
- 二次探测 (quadratic probing)：负载因子小于 1，按照二次方程作为下一个位置的偏移逐一向下寻找 (两个元素经 hash 函数计算出的位置相同，那么之后插入时探测的位置也相同，导致浪费)
- 链地址 (separate chaining)：负载因子大于 1，在每个 hash table 元素上维护一个链表

> 如果底层桶个数的大小为 **质数**，并且保持负载因子在 **0.5** 以下，那么可以确定每插入一个新元素所需探测次数不多于 2 (为什么？)。当桶的个数需要增长时，需要找到比当前桶个数大两倍左右的质数。

## 5.7.2 hashtable 的桶 (bucket) 与结点 (nodes)

桶是 hash table 中的单元，每个桶中包含了若干结点。桶结点的定义如下：

```c++
template <class _Val>
struct _Hashtable_node
{
  _Hashtable_node* _M_next;
  _Val _M_val;
};
```

桶维护的链表并没有使用 STL 的 list 或 slist，而是自行维护。所有的桶结点被维护在一个 vector 内，以便动态扩充。

## 5.7.3 hashtable 的迭代器

```c++
template <class _Val, class _Key, class _HashFcn,
          class _ExtractKey, class _EqualKey, class _Alloc>
struct _Hashtable_iterator {
  typedef hashtable<_Val,_Key,_HashFcn,_ExtractKey,_EqualKey,_Alloc>
          _Hashtable;
  typedef _Hashtable_iterator<_Val, _Key, _HashFcn,
                              _ExtractKey, _EqualKey, _Alloc>
          iterator;
  typedef _Hashtable_const_iterator<_Val, _Key, _HashFcn,
                                    _ExtractKey, _EqualKey, _Alloc>
          const_iterator;
  typedef _Hashtable_node<_Val> _Node;

  typedef forward_iterator_tag iterator_category; // 单向迭代器
  typedef _Val value_type;
  typedef ptrdiff_t difference_type;
  typedef size_t size_type;
  typedef _Val& reference;
  typedef _Val* pointer;

  _Node* _M_cur;     // 指向结点
  _Hashtable* _M_ht; // 指向 hashtable

  _Hashtable_iterator(_Node* __n, _Hashtable* __tab)
    : _M_cur(__n), _M_ht(__tab) {}
  _Hashtable_iterator() {}
  reference operator*() const { return _M_cur->_M_val; }
#ifndef __SGI_STL_NO_ARROW_OPERATOR
  pointer operator->() const { return &(operator*()); }
#endif /* __SGI_STL_NO_ARROW_OPERATOR */
  iterator& operator++();
  iterator operator++(int);
  bool operator==(const iterator& __it) const
    { return _M_cur == __it._M_cur; }
  bool operator!=(const iterator& __it) const
    { return _M_cur != __it._M_cur; }
};
```

hashtable 的迭代器必须维护与 hashtable 的联系。迭代器的前进操作意味着利用结点的 `next` 指针访问桶内链表的下一个元素；如果当前结点刚好是链表的尾端，那么就应当跳转到下一个 bucket 上。

```c++
template <class _Val, class _Key, class _HF, class _ExK, class _EqK,
          class _All>
_Hashtable_iterator<_Val,_Key,_HF,_ExK,_EqK,_All>&
_Hashtable_iterator<_Val,_Key,_HF,_ExK,_EqK,_All>::operator++()
{
  const _Node* __old = _M_cur; // 当前结点
  _M_cur = _M_cur->_M_next;    // 当前结点的下一个结点
  if (!_M_cur) { // 下一个结点已经是链表尾
    size_type __bucket = _M_ht->_M_bkt_num(__old->_M_val);    // 当前结点所在的桶
    while (!_M_cur && ++__bucket < _M_ht->_M_buckets.size())  // 桶为空，寻找下一个桶
      _M_cur = _M_ht->_M_buckets[__bucket];                   // 下一个非空桶的第一个元素
  }
  return *this;
}

template <class _Val, class _Key, class _HF, class _ExK, class _EqK,
          class _All>
inline _Hashtable_iterator<_Val,_Key,_HF,_ExK,_EqK,_All>
_Hashtable_iterator<_Val,_Key,_HF,_ExK,_EqK,_All>::operator++(int)
{
  iterator __tmp = *this; // 暂存返回结点
  ++*this;                // 自增
  return __tmp;           // 返回自增前的结点
}
```

## 5.7.4 hashtable 的数据结构

```c++
template <class _Val, class _Key, class _HashFcn,
          class _ExtractKey, class _EqualKey, class _Alloc>
class hashtable {
public:
  typedef _Key key_type;
  typedef _Val value_type;
  typedef _HashFcn hasher;
  typedef _EqualKey key_equal;

  typedef size_t            size_type;
  typedef ptrdiff_t         difference_type;
  typedef value_type*       pointer;
  typedef const value_type* const_pointer;
  typedef value_type&       reference;
  typedef const value_type& const_reference;

  hasher hash_funct() const { return _M_hash; }
  key_equal key_eq() const { return _M_equals; }

private:
  typedef _Hashtable_node<_Val> _Node;

public:
  typedef _Alloc allocator_type;
  allocator_type get_allocator() const { return allocator_type(); }
private:
  typedef simple_alloc<_Node, _Alloc> _M_node_allocator_type; // 以结点大小为单位分配结点
  _Node* _M_get_node() { return _M_node_allocator_type::allocate(1); }
  void _M_put_node(_Node* __p) { _M_node_allocator_type::deallocate(__p, 1); }

private:
  hasher                _M_hash;    // 计算 hash 的函数 (仿函数)
  key_equal             _M_equals;  // 比较 key 的函数 (仿函数)
  _ExtractKey           _M_get_key; // 从结点中得到 key 的函数 (仿函数)
  vector<_Node*,_Alloc> _M_buckets; // 桶数组
  size_type             _M_num_elements;

public:
  // ...
};
```

链地址法并不需要桶数组的大小为质数，但 SGI STL 仍然以质数来设置桶数组的大小。STL 准备了 28 个接近两倍的质数，并提供一个函数，用于查找最接近并大于等于指定值的那个质数：

```c++
// Note: assumes long is at least 32 bits.
enum { __stl_num_primes = 28 };

static const unsigned long __stl_prime_list[__stl_num_primes] =
{
  53ul,         97ul,         193ul,       389ul,       769ul,
  1543ul,       3079ul,       6151ul,      12289ul,     24593ul,
  49157ul,      98317ul,      196613ul,    393241ul,    786433ul,
  1572869ul,    3145739ul,    6291469ul,   12582917ul,  25165843ul,
  50331653ul,   100663319ul,  201326611ul, 402653189ul, 805306457ul,
  1610612741ul, 3221225473ul, 4294967291ul
};

inline unsigned long __stl_next_prime(unsigned long __n)
{
  const unsigned long* __first = __stl_prime_list;
  const unsigned long* __last = __stl_prime_list + (int)__stl_num_primes;
  const unsigned long* pos = lower_bound(__first, __last, __n); // 二分查找
  return pos == __last ? *(__last - 1) : *pos;
}
```

与桶数组尺寸相关的 API：

```c++
size_type bucket_count() const { return _M_buckets.size(); }

size_type max_bucket_count() const
  { return __stl_prime_list[(int)__stl_num_primes - 1]; }
```

## 5.7.5 hashtable 的构造与内存管理

通过分配器获得一个结点的空间，构造该结点并返回：

```c++
typedef simple_alloc<_Node, _Alloc> _M_node_allocator_type;
_Node* _M_get_node() { return _M_node_allocator_type::allocate(1); }
void _M_put_node(_Node* __p) { _M_node_allocator_type::deallocate(__p, 1); }

_Node* _M_new_node(const value_type& __obj)
{
    _Node* __n = _M_get_node();
    __n->_M_next = 0;
    __STL_TRY {
        construct(&__n->_M_val, __obj);
        return __n;
    }
    __STL_UNWIND(_M_put_node(__n));
}

void _M_delete_node(_Node* __n)
{
    destroy(&__n->_M_val);
    _M_put_node(__n);
}
```

初始化桶数组，默认将桶内的结点指针全部设为空：

```c++
void _M_initialize_buckets(size_type __n)
{
    const size_type __n_buckets = _M_next_size(__n); // 以大于等于 n 的第一个质数作为桶容量
    _M_buckets.reserve(__n_buckets);                 // 保留桶容量大小的空间
    _M_buckets.insert(_M_buckets.end(), __n_buckets, (_Node*) 0); // 插入值为 0 的元素 (空指针)
    _M_num_elements = 0; // 桶内元素为 0 个
}

hashtable(size_type __n,
          const _HashFcn&    __hf,
          const _EqualKey&   __eql,
          const _ExtractKey& __ext,
          const allocator_type& __a = allocator_type())
    : __HASH_ALLOC_INIT(__a)
        _M_hash(__hf),
        _M_equals(__eql),
        _M_get_key(__ext),
        _M_buckets(__a),
        _M_num_elements(0)
{
    _M_initialize_buckets(__n);
}
```

不允许重复的元素插入操作过程：

```c++
pair<iterator, bool> insert_unique(const value_type& __obj)
{
    resize(_M_num_elements + 1);           // 元素个数增加，判断是否需要 rehash
    return insert_unique_noresize(__obj);  // 不 rehash，直接插入元素
}

template <class _Val, class _Key, class _HF, class _Ex, class _Eq, class _All>
void hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>
  ::resize(size_type __num_elements_hint)
{
  const size_type __old_n = _M_buckets.size(); // 桶个数
  if (__num_elements_hint > __old_n) {         // 元素数量超过了桶个数
    const size_type __n = _M_next_size(__num_elements_hint); // 寻找扩容后的桶个数
    if (__n > __old_n) {
      vector<_Node*, _All> __tmp(__n, (_Node*)(0),
                                 _M_buckets.get_allocator()); // 分配新的桶空间
      __STL_TRY {
        for (size_type __bucket = 0; __bucket < __old_n; ++__bucket) {
          _Node* __first = _M_buckets[__bucket]; // 旧桶内的第一个元素
          while (__first) {
            size_type __new_bucket = _M_bkt_num(__first->_M_val, __n); // 计算新的 hash 值
            _M_buckets[__bucket] = __first->_M_next; // 将 first 从旧桶链表中取出
            __first->_M_next = __tmp[__new_bucket];  // first 进入新桶链表的第一个位置
            __tmp[__new_bucket] = __first;
            __first = _M_buckets[__bucket];          // 旧桶链表的 (新) 头元素
          }
        }
        _M_buckets.swap(__tmp); // 将临时的新桶与旧桶交换，旧桶将会随着函数退出而被析构
      }
#         ifdef __STL_USE_EXCEPTIONS
      catch(...) { // 异常发生，回滚
        for (size_type __bucket = 0; __bucket < __tmp.size(); ++__bucket) {
          while (__tmp[__bucket]) {
            _Node* __next = __tmp[__bucket]->_M_next; // 指向临时桶的第二个结点
            _M_delete_node(__tmp[__bucket]);          // 销毁临时桶的第一个结点
            __tmp[__bucket] = __next;                 // 临时桶的原第二个结点成为第一个结点
          }
        }
        throw;
      }
#         endif /* __STL_USE_EXCEPTIONS */
    }
  }
}

template <class _Val, class _Key, class _HF, class _Ex, class _Eq, class _All>
pair<typename hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>::iterator, bool>
hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>
  ::insert_unique_noresize(const value_type& __obj)
{
  const size_type __n = _M_bkt_num(__obj); // 计算出元素的 hash
  _Node* __first = _M_buckets[__n];        // 元素 hash 对应的桶的第一个元素

  for (_Node* __cur = __first; __cur; __cur = __cur->_M_next) // 遍历桶链表
    if (_M_equals(_M_get_key(__cur->_M_val), _M_get_key(__obj))) // 发现相等元素
      return pair<iterator, bool>(iterator(__cur, this), false); // 插入失败

  _Node* __tmp = _M_new_node(__obj); // 构造新结点
  __tmp->_M_next = __first;          // 将新结点作为桶链表的第一个结点
  _M_buckets[__n] = __tmp;
  ++_M_num_elements;                 // 元素个数++
  return pair<iterator, bool>(iterator(__tmp, this), true); // 插入成功
}
```

允许重复的元素插入操作过程：

```c++
iterator insert_equal(const value_type& __obj) // 由于操作肯定成功，因此不返回 pair
{
    resize(_M_num_elements + 1); // 同上
    return insert_equal_noresize(__obj);
}

template <class _Val, class _Key, class _HF, class _Ex, class _Eq, class _All>
typename hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>::iterator
hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>
  ::insert_equal_noresize(const value_type& __obj)
{
  const size_type __n = _M_bkt_num(__obj); // 计算元素 hash
  _Node* __first = _M_buckets[__n];        // 元素所在的桶链表

  for (_Node* __cur = __first; __cur; __cur = __cur->_M_next) // 遍历桶链表
    if (_M_equals(_M_get_key(__cur->_M_val), _M_get_key(__obj))) { // 如果相等
      _Node* __tmp = _M_new_node(__obj); // 分配新结点
      __tmp->_M_next = __cur->_M_next;   // 在当前位置立刻插入
      __cur->_M_next = __tmp;
      ++_M_num_elements;                 // 元素个数++
      return iterator(__tmp, this);      // 返回指向新结点的迭代器
    }

  // 没有找到任何相等的结点
  _Node* __tmp = _M_new_node(__obj); // 分配新结点
  __tmp->_M_next = __first;          // 新结点插入在桶链表的头部
  _M_buckets[__n] = __tmp;
  ++_M_num_elements;                 // 元素个数++
  return iterator(__tmp, this);      // 返回指向新结点的迭代器
}
```

判断元素所属的桶。由 hash 函数完成。如果不提供用于取模的数量，则使用桶的容量：

```c++
size_type _M_bkt_num_key(const key_type& __key) const
{
    return _M_bkt_num_key(__key, _M_buckets.size());
}

size_type _M_bkt_num(const value_type& __obj) const
{
    return _M_bkt_num_key(_M_get_key(__obj));
}

size_type _M_bkt_num_key(const key_type& __key, size_t __n) const
{
    return _M_hash(__key) % __n;
}

size_type _M_bkt_num(const value_type& __obj, size_t __n) const
{
    return _M_bkt_num_key(_M_get_key(__obj), __n);
}
```

整体删除。需要注意内存释放的问题：

```c++
template <class _Val, class _Key, class _HF, class _Ex, class _Eq, class _All>
void hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>::clear()
{
  for (size_type __i = 0; __i < _M_buckets.size(); ++__i) { // 遍历每一个桶
    _Node* __cur = _M_buckets[__i];
    while (__cur != 0) {
      _Node* __next = __cur->_M_next; // 当前结点之后的结点
      _M_delete_node(__cur);          // 删除当前结点
      __cur = __next;                 // 继续之后的结点
    }
    _M_buckets[__i] = 0; // 桶指针置空
  }
  _M_num_elements = 0;   // 元素个数清零
} // 没有释放桶对应的 vector 空间
```

从另一个 hashtable 复制：

```c++
template <class _Val, class _Key, class _HF, class _Ex, class _Eq, class _All>
void hashtable<_Val,_Key,_HF,_Ex,_Eq,_All>
  ::_M_copy_from(const hashtable& __ht)
{
  _M_buckets.clear(); // 先将当前桶中的元素清空 (容量保留)
  _M_buckets.reserve(__ht._M_buckets.size()); // 将当前桶容量扩容至与复制目标一致
  _M_buckets.insert(_M_buckets.end(), __ht._M_buckets.size(), (_Node*) 0); // 将桶容量补齐 (空指针)
  __STL_TRY {
    for (size_type __i = 0; __i < __ht._M_buckets.size(); ++__i) { // 复制目标中的每一个桶
      const _Node* __cur = __ht._M_buckets[__i]; // 桶内链表的第一个元素
      if (__cur) { // 桶不为空
        _Node* __copy = _M_new_node(__cur->_M_val); // 复制第一个结点
        _M_buckets[__i] = __copy;                   // 加入链表

        // 复制后续结点
        for (_Node* __next = __cur->_M_next;
             __next;
             __cur = __next, __next = __cur->_M_next) {
          __copy->_M_next = _M_new_node(__next->_M_val);
          __copy = __copy->_M_next;
        }
      }
    }
    _M_num_elements = __ht._M_num_elements; // 设置元素个数
  }
  __STL_UNWIND(clear()); // 失败回滚，将元素全部清空
}
```

## 5.7.7 hash functions

hash 函数的主要工作是将数据转换成一个可以进行模运算的值。对于整数类型 (`int` / `char` / `long`) 来说，大部分 hash 函数什么都不做，直接返回原值：

```c++
__STL_TEMPLATE_NULL struct hash<char> {
  size_t operator()(char __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned char> {
  size_t operator()(unsigned char __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<signed char> {
  size_t operator()(unsigned char __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<short> {
  size_t operator()(short __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned short> {
  size_t operator()(unsigned short __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<int> {
  size_t operator()(int __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned int> {
  size_t operator()(unsigned int __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<long> {
  size_t operator()(long __x) const { return __x; }
};
__STL_TEMPLATE_NULL struct hash<unsigned long> {
  size_t operator()(unsigned long __x) const { return __x; }
};
```

对于字符串来说，STL 设计了一个转换函数：

```c++
inline size_t __stl_hash_string(const char* __s)
{
  unsigned long __h = 0;
  for ( ; *__s; ++__s)
    __h = 5*__h + *__s;

  return size_t(__h);
}

__STL_TEMPLATE_NULL struct hash<char*>
{
  size_t operator()(const char* __s) const { return __stl_hash_string(__s); }
};

__STL_TEMPLATE_NULL struct hash<const char*>
{
  size_t operator()(const char* __s) const { return __stl_hash_string(__s); }
};
```

SGI STL 无法处理上述所列各数据类型以外的元素。如果要处理这些类型，用户需要为它们自行定义 hash 函数。
