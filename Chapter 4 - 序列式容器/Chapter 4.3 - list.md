# Chapter 4.3 - list

Created by : Mr Dk.

2021 / 04 / 03 21:40

Nanjing, Jiangsu, China

---

## 4.3.1 概述

单个头结点的环形双向链表设计，好处是每次插入或删除元素就分配或释放一个元素的空间，因此对空间的运用绝对精准，一点也不浪费；对任何位置元素的插入和删除永远是常数时间。

## 4.3.2 list 的结点 (node)

双向链表，除了数据以外，还有两个前后指针：

```c++
struct _List_node_base {
  _List_node_base* _M_next;
  _List_node_base* _M_prev;
};

template <class _Tp>
struct _List_node : public _List_node_base {
  _Tp _M_data;
};
```

## 4.3.3 list 的迭代器

list 的结点并不在内存中连续存储，因此不能像 vector 一样使用普通指针作为迭代器。list 的迭代器应当能够在递增 / 递减时正确指向下一个 / 前一个元素。因此，list 提供 Bidirectional Iterators。

```c++
inline bidirectional_iterator_tag
iterator_category(const _List_iterator_base&)
{
  return bidirectional_iterator_tag();
}
```

list 迭代器的重要性质是，insert 或 splice 操作都不会使原有迭代器失效；erase 操作除去被删除结点以外的迭代器也不会失效。

list 的迭代器内需要维护一个指向结点的指针。通过该指针，能够访问到 list 结点。对迭代器的自增 / 自减能够使迭代器指向下一个 / 上一个 list 结点。

```c++
struct _List_iterator_base {
  typedef size_t                     size_type;
  typedef ptrdiff_t                  difference_type;
  typedef bidirectional_iterator_tag iterator_category;

  _List_node_base* _M_node;

  _List_iterator_base(_List_node_base* __x) : _M_node(__x) {}
  _List_iterator_base() {}

  void _M_incr() { _M_node = _M_node->_M_next; }
  void _M_decr() { _M_node = _M_node->_M_prev; }

  bool operator==(const _List_iterator_base& __x) const {
    return _M_node == __x._M_node;
  }
  bool operator!=(const _List_iterator_base& __x) const {
    return _M_node != __x._M_node;
  }
};  

template<class _Tp, class _Ref, class _Ptr>
struct _List_iterator : public _List_iterator_base {
  typedef _List_iterator<_Tp,_Tp&,_Tp*>             iterator;
  typedef _List_iterator<_Tp,const _Tp&,const _Tp*> const_iterator;
  typedef _List_iterator<_Tp,_Ref,_Ptr>             _Self;

  typedef _Tp value_type;
  typedef _Ptr pointer;
  typedef _Ref reference;
  typedef _List_node<_Tp> _Node;

  _List_iterator(_Node* __x) : _List_iterator_base(__x) {}
  _List_iterator() {}
  _List_iterator(const iterator& __x) : _List_iterator_base(__x._M_node) {}

  reference operator*() const { return ((_Node*) _M_node)->_M_data; }

#ifndef __SGI_STL_NO_ARROW_OPERATOR
  pointer operator->() const { return &(operator*()); }
#endif /* __SGI_STL_NO_ARROW_OPERATOR */

  _Self& operator++() { 
    this->_M_incr();
    return *this;
  }
  _Self operator++(int) { 
    _Self __tmp = *this;
    this->_M_incr();
    return __tmp;
  }
  _Self& operator--() { 
    this->_M_decr();
    return *this;
  }
  _Self operator--(int) { 
    _Self __tmp = *this;
    this->_M_decr();
    return __tmp;
  }
};
```

## 4.3.4 list 的数据结构

由于 SGI list 是一个环形双向链表，因此只需要一个指针就能表示整个链表。该指针可以指向一个 dummy 的空白结点，从而能够完成 STL **左闭右开** 的区间要求。

```c++
template <class _Tp, class _Alloc>
class _List_base 
{
public:
  typedef _Alloc allocator_type;
  allocator_type get_allocator() const { return allocator_type(); }

  _List_base(const allocator_type&) {
    _M_node = _M_get_node();
    _M_node->_M_next = _M_node;
    _M_node->_M_prev = _M_node;
  }
  ~_List_base() {
    clear();
    _M_put_node(_M_node);
  }

  void clear();

protected:
  typedef simple_alloc<_List_node<_Tp>, _Alloc> _Alloc_type;
  _List_node<_Tp>* _M_get_node() { return _Alloc_type::allocate(1); }
  void _M_put_node(_List_node<_Tp>* __p) { _Alloc_type::deallocate(__p, 1); } 

protected:
  _List_node<_Tp>* _M_node;
};
```

初始状态下，`node` 指针将会指向一个空白结点，该结点的 `prev` 和 `next` 都指向自身。通过 `node` 指针，以下成员函数可以轻松实现：

```c++
iterator begin()             { return (_Node*)(_M_node->_M_next); }
const_iterator begin() const { return (_Node*)(_M_node->_M_next); }

iterator end()             { return _M_node; }
const_iterator end() const { return _M_node; }

bool empty() const { return _M_node->_M_next == _M_node; }
size_type size() const {
    size_type __result = 0;
    distance(begin(), end(), __result);
    return __result;
}
size_type max_size() const { return size_type(-1); }

reference front() { return *begin(); }
const_reference front() const { return *begin(); }
reference back() { return *(--end()); }
const_reference back() const { return *(--end()); }
```

## 4.3.5 list 的构造于内存管理：constructor，push_back，insert

list 使用缺省的 `alloc` 空间分配器，并在类内定义了一个分配器，方便以 list 结点大小为单位分配内存：

```c++
typedef simple_alloc<_List_node<_Tp>, _Alloc> _Alloc_type;
_List_node<_Tp>* _M_get_node() { return _Alloc_type::allocate(1); }
void _M_put_node(_List_node<_Tp>* __p) { _Alloc_type::deallocate(__p, 1); } 
```

以下两个子函数除了分配内存，还调用构造函数构造对象：

```c++
_Node* _M_create_node(const _Tp& __x)
{
    _Node* __p = _M_get_node();
    __STL_TRY {
        _Construct(&__p->_M_data, __x);
    }
    __STL_UNWIND(_M_put_node(__p));
    return __p;
}

_Node* _M_create_node()
{
    _Node* __p = _M_get_node();
    __STL_TRY {
        _Construct(&__p->_M_data);
    }
    __STL_UNWIND(_M_put_node(__p));
    return __p;
}
```

list 的插入函数 `insert()` 有多种重载，但本质上都需要确定一个插入位置。元素将会被插入到该位置之前：该位置的元素将成为所有被插入元素之后的第一个元素，这也是 STL 的插入规范。其余的，就是分配空间与构造结点的过程。

```c++
iterator insert(iterator __position, const _Tp& __x) {
    _Node* __tmp = _M_create_node(__x);
    __tmp->_M_next = __position._M_node;
    __tmp->_M_prev = __position._M_node->_M_prev;
    __position._M_node->_M_prev->_M_next = __tmp;
    __position._M_node->_M_prev = __tmp;
    return __tmp;
}
```

有了这个标准实现，其它的 `insert()` 无非就是指定不同的插入位置罢了。包括 `push_back()` 和 `push_front()`：

```c++
void push_front(const _Tp& __x) { insert(begin(), __x); }
void push_front() {insert(begin());}
void push_back(const _Tp& __x) { insert(end(), __x); }
void push_back() {insert(end());}
```

区间插入也是同样道理。

## 4.3.6 list 的元素操作

`erase()` 操作与 `insert()` 相反。将迭代器指向的结点或结点区间从链表中取出来 (并重新修复链表内的指针) 后，依次调用析构与内存方式即可。

```c++
iterator erase(iterator __position) {
    _List_node_base* __next_node = __position._M_node->_M_next;
    _List_node_base* __prev_node = __position._M_node->_M_prev;
    _Node* __n = (_Node*) __position._M_node;
    __prev_node->_M_next = __next_node;
    __next_node->_M_prev = __prev_node;
    _Destroy(&__n->_M_data);
    _M_put_node(__n);
    return iterator((_Node*) __next_node);
}

iterator erase(iterator __first, iterator __last)
{
  while (__first != __last)
    erase(__first++);
  return __last;
}
```

基于此，`pop_back()` 和 `pop_front()` 也可以被实现了：

```c++
void pop_front() { erase(begin()); }
void pop_back() { 
    iterator __tmp = end();
    erase(--__tmp);
}
```

`remove()` 删除所有特定值的结点：

```c++
template <class _Tp, class _Alloc>
void list<_Tp, _Alloc>::remove(const _Tp& __value)
{
  iterator __first = begin();
  iterator __last = end();
  while (__first != __last) {
    iterator __next = __first;
    ++__next;
    if (*__first == __value) erase(__first);
    __first = __next;
  }
}
```

`unique()` 移除 **数值相同且连续** 的元素，移除至只剩下一个：

```c++
template <class _Tp, class _Alloc>
void list<_Tp, _Alloc>::unique()
{
  iterator __first = begin();
  iterator __last = end();
  if (__first == __last) return;
  iterator __next = __first;
  while (++__next != __last) { // next 指针前进
    if (*__first == *__next) // first 指针不前进
      erase(__next);         // 删除 next 指针指向的结点
    else
      __first = __next;      // first 指针前进
    __next = __first;
  }
}
```

`tranfer()` 提供了 **区间插入操作**：将某个连续范围内的元素迁移到某个特定位置之前。在实现上，只需要修改指针即可。将 `first` 指向的结点续到插入位置之前的结点之后；从 `last` 结点的后面接上原插入位置开始的结点。

> 该函数接受插入区间和插入位置在同一个 list 中。

```c++
void transfer(iterator __position, iterator __first, iterator __last) {
    if (__position != __last) {
        // Remove [first, last) from its old position.
        __last._M_node->_M_prev->_M_next     = __position._M_node;
        __first._M_node->_M_prev->_M_next    = __last._M_node;
        __position._M_node->_M_prev->_M_next = __first._M_node; 

        // Splice [first, last) into its new position.
        _List_node_base* __tmp      = __position._M_node->_M_prev;
        __position._M_node->_M_prev = __last._M_node->_M_prev;
        __last._M_node->_M_prev     = __first._M_node->_M_prev; 
        __first._M_node->_M_prev    = __tmp;
    }
}
```

上述函数并不是 `public` 的，而是为其它函数的实现奠定了基础。

`splice()` 函数能够将一个区间插入到 list 迭代器指示的位置：

```c++
void splice(iterator __position, list& __x) { // x 与当前 list 不能是同一个 list
    if (!__x.empty()) 
        this->transfer(__position, __x.begin(), __x.end());
}
void splice(iterator __position, list&, iterator __i) { // 仅插入 i 指向的结点
    iterator __j = __i;
    ++__j;
    if (__position == __i || __position == __j) return;
    this->transfer(__position, __i, __j);
}
void splice(iterator __position, list&, iterator __first, iterator __last) {
    if (__first != __last) 
        this->transfer(__position, __first, __last);
}
```

`merge()` 将两个 **已经递增排序** 的 list 合并到调用该函数的 list 身上：

```c++
template <class _Tp, class _Alloc>
void list<_Tp, _Alloc>::merge(list<_Tp, _Alloc>& __x)
{
  iterator __first1 = begin();
  iterator __last1 = end();
  iterator __first2 = __x.begin();
  iterator __last2 = __x.end();
  while (__first1 != __last1 && __first2 != __last2)
    if (*__first2 < *__first1) {
      iterator __next = __first2;
      transfer(__first1, __first2, ++__next); // 将 list2 中的结点转移到 list1 (*this)
      __first2 = __next;
    }
    else
      ++__first1;
  if (__first2 != __last2) transfer(__last1, __first2, __last2);
}
```

提到排序，list 不能使用 STL 的泛化版 `sort()`，因为泛化算法只接受 Random Access Iterators。list 类实现了自己的 `sort()`：据称这是一个 **快速排序**，但我看不懂...... 😭

```c++
template <class _Tp, class _Alloc>
void list<_Tp, _Alloc>::sort()
{
  // Do nothing if the list has length 0 or 1.
  if (_M_node->_M_next != _M_node && _M_node->_M_next->_M_next != _M_node) {
    list<_Tp, _Alloc> __carry;
    list<_Tp, _Alloc> __counter[64];
    int __fill = 0;
    while (!empty()) {
      __carry.splice(__carry.begin(), *this, begin());
      int __i = 0;
      while(__i < __fill && !__counter[__i].empty()) {
        __counter[__i].merge(__carry);
        __carry.swap(__counter[__i++]);
      }
      __carry.swap(__counter[__i]);         
      if (__i == __fill) ++__fill;
    } 

    for (int __i = 1; __i < __fill; ++__i)
      __counter[__i].merge(__counter[__i-1]);
    swap(__counter[__fill-1]);
  }
}
```

`reverse()` 函数将 list 中的所有元素逆置。具体实现是，将每个元素从链表尾 tranfer 到链表头。但我看到的版本是这样的：

```c++
inline void __List_base_reverse(_List_node_base* __p)
{
  _List_node_base* __tmp = __p;
  do {
    __STD::swap(__tmp->_M_next, __tmp->_M_prev);
    __tmp = __tmp->_M_prev;     // Old next node is now prev.
  } while (__tmp != __p);
}

template <class _Tp, class _Alloc>
inline void list<_Tp, _Alloc>::reverse() 
{
  __List_base_reverse(this->_M_node);
}    
```

---

