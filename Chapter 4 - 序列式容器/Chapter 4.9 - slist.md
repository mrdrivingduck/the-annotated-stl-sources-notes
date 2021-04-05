# Chapter 4.9 - slist

Created by : Mr Dk.

2021 / 04 / 05 11:22

Nanjing, Jiangsu, China

---

## 4.9.1 slist 概述

STL 的 list 是双向链表。SGI STL 还另外提供了一个单向链表 slist。该容器不在标准规格以内。slist 的迭代器是单项的 forward iterator，而 list 的迭代器是双向的 bidirectional iterator。因此，slist 的功能受到了许多限制。它们的共同点是，对容器进行修改操作不太容易使迭代器失效 (因为只是修改了部分指针)。

根据 STL 的习惯，插入操作发生在迭代器指定的结点之前，而由于单向链表没有办法获得前一个结点的引用，因此，slist 需要从头开始寻找插入位置。也就是说，除了在 slist 起点附近，调用 `insert()` 或 `erase()` 都是不智的。根据 slist 的特性，slist 特别提供了 `insert_after()` 和 `erase_after()`。另外，出于效率考虑，slist 只提供 `push_front()`，不提供 `push_back()`。

## 4.9.2 slist 的结点

结点运用继承关系定义：

```c++
struct _Slist_node_base
{
  _Slist_node_base* _M_next;
};

template <class _Tp>
struct _Slist_node : public _Slist_node_base
{
  _Tp _M_data;
};
```

链接新结点：

```c++
inline _Slist_node_base*
__slist_make_link(_Slist_node_base* __prev_node,
                  _Slist_node_base* __new_node)
{
  __new_node->_M_next = __prev_node->_M_next;
  __prev_node->_M_next = __new_node;
  return __new_node;
}
```

获得某个结点的前一个结点 (需要从头开始找)：

```c++
inline _Slist_node_base* 
__slist_previous(_Slist_node_base* __head,
                 const _Slist_node_base* __node)
{
  while (__head && __head->_M_next != __node)
    __head = __head->_M_next;
  return __head;
}

inline const _Slist_node_base* 
__slist_previous(const _Slist_node_base* __head,
                 const _Slist_node_base* __node)
{
  while (__head && __head->_M_next != __node)
    __head = __head->_M_next;
  return __head;
}
```

获取结点个数 (从前开始找)：

```c++
inline size_t __slist_size(_Slist_node_base* __node)
{
  size_t __result = 0;
  for ( ; __node != 0; __node = __node->_M_next)
    ++__result;
  return __result;
}
```

## 4.9.3 slist 的迭代器

内部维护了一个指向结点的指针。

```c++
struct _Slist_iterator_base
{
  typedef size_t               size_type;
  typedef ptrdiff_t            difference_type;
  typedef forward_iterator_tag iterator_category; // 单向迭代器

  _Slist_node_base* _M_node; // 指向结点的指针

  _Slist_iterator_base(_Slist_node_base* __x) : _M_node(__x) {}
  void _M_incr() { _M_node = _M_node->_M_next; }

  bool operator==(const _Slist_iterator_base& __x) const {
    return _M_node == __x._M_node;
  }
  bool operator!=(const _Slist_iterator_base& __x) const {
    return _M_node != __x._M_node;
  }
};

template <class _Tp, class _Ref, class _Ptr>
struct _Slist_iterator : public _Slist_iterator_base
{
  typedef _Slist_iterator<_Tp, _Tp&, _Tp*>             iterator;
  typedef _Slist_iterator<_Tp, const _Tp&, const _Tp*> const_iterator;
  typedef _Slist_iterator<_Tp, _Ref, _Ptr>             _Self;

  typedef _Tp              value_type;
  typedef _Ptr             pointer;
  typedef _Ref             reference;
  typedef _Slist_node<_Tp> _Node;

  _Slist_iterator(_Node* __x) : _Slist_iterator_base(__x) {}
  _Slist_iterator() : _Slist_iterator_base(0) {}
  _Slist_iterator(const iterator& __x) : _Slist_iterator_base(__x._M_node) {}

  reference operator*() const { return ((_Node*) _M_node)->_M_data; }
  pointer operator->() const { return &(operator*()); }

  _Self& operator++()
  {
    _M_incr();
    return *this;
  }
  _Self operator++(int)
  {
    _Self __tmp = *this;
    _M_incr();
    return __tmp;
  }
};
```

## 4.9.4 slist 的数据结构

slist 内维护了一个实物头结点。

```c++
template <class _Tp, class _Alloc> 
struct _Slist_base {
  typedef _Alloc allocator_type;
  allocator_type get_allocator() const { return allocator_type(); }

  _Slist_base(const allocator_type&) { _M_head._M_next = 0; }
  ~_Slist_base() { _M_erase_after(&_M_head, 0); }

protected:
  typedef simple_alloc<_Slist_node<_Tp>, _Alloc> _Alloc_type; // 以结点为单位分配内存
  _Slist_node<_Tp>* _M_get_node() { return _Alloc_type::allocate(1); }
  void _M_put_node(_Slist_node<_Tp>* __p) { _Alloc_type::deallocate(__p, 1); }

  _Slist_node_base* _M_erase_after(_Slist_node_base* __pos)
  {
    _Slist_node<_Tp>* __next = (_Slist_node<_Tp>*) (__pos->_M_next);
    _Slist_node_base* __next_next = __next->_M_next;
    __pos->_M_next = __next_next; // 取下结点
    destroy(&__next->_M_data);    // 析构结点
    _M_put_node(__next);          // 释放结点空间
    return __next_next;
  }
  _Slist_node_base* _M_erase_after(_Slist_node_base*, _Slist_node_base*);

protected:
  _Slist_node_base _M_head; // 头结点 (不是结点指针，而就是一个结点)
}; 
```

删除一个范围以内的结点 (由范围之前的一个结点和范围内最后一个结点的下一个结点指示)：

```c++
template <class _Tp, class _Alloc> 
_Slist_node_base*
_Slist_base<_Tp,_Alloc>::_M_erase_after(_Slist_node_base* __before_first,
                                        _Slist_node_base* __last_node) {
  _Slist_node<_Tp>* __cur = (_Slist_node<_Tp>*) (__before_first->_M_next);
  while (__cur != __last_node) {
    _Slist_node<_Tp>* __tmp = __cur;
    __cur = (_Slist_node<_Tp>*) __cur->_M_next;
    destroy(&__tmp->_M_data);
    _M_put_node(__tmp);
  }
  __before_first->_M_next = __last_node; // before_first 和 last_node 都被保留了
  return __last_node;
}
```

slist 主类：

```c++
template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
class slist : private _Slist_base<_Tp,_Alloc>
{
  // requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);

private:
  typedef _Slist_base<_Tp,_Alloc> _Base;
public:
  typedef _Tp                value_type;
  typedef value_type*       pointer;
  typedef const value_type* const_pointer;
  typedef value_type&       reference;
  typedef const value_type& const_reference;
  typedef size_t            size_type;
  typedef ptrdiff_t         difference_type;

  typedef _Slist_iterator<_Tp, _Tp&, _Tp*>             iterator;
  typedef _Slist_iterator<_Tp, const _Tp&, const _Tp*> const_iterator;

  typedef typename _Base::allocator_type allocator_type;
  allocator_type get_allocator() const { return _Base::get_allocator(); }

private:
  typedef _Slist_node<_Tp>      _Node;
  typedef _Slist_node_base      _Node_base;
  typedef _Slist_iterator_base  _Iterator_base;

  _Node* _M_create_node(const value_type& __x) {
    _Node* __node = this->_M_get_node(); // 分配结点空间
    __STL_TRY {
      construct(&__node->_M_data, __x);  // 拷贝构造结点
      __node->_M_next = 0;
    }
    __STL_UNWIND(this->_M_put_node(__node));
    return __node;
  }
  
  _Node* _M_create_node() {
    _Node* __node = this->_M_get_node(); // 分配结点空间
    __STL_TRY {
      construct(&__node->_M_data);       // 拷贝构造结点
      __node->_M_next = 0;
    }
    __STL_UNWIND(this->_M_put_node(__node));
    return __node;
  }

public:
  explicit slist(const allocator_type& __a = allocator_type()) : _Base(__a) {}

  slist(size_type __n, const value_type& __x,
        const allocator_type& __a =  allocator_type()) : _Base(__a)
    { _M_insert_after_fill(&this->_M_head, __n, __x); }

  explicit slist(size_type __n) : _Base(allocator_type())
    { _M_insert_after_fill(&this->_M_head, __n, value_type()); }

  slist(const_iterator __first, const_iterator __last,
        const allocator_type& __a =  allocator_type()) : _Base(__a)
    { _M_insert_after_range(&this->_M_head, __first, __last); }
  slist(const value_type* __first, const value_type* __last,
        const allocator_type& __a =  allocator_type()) : _Base(__a)
    { _M_insert_after_range(&this->_M_head, __first, __last); }

  slist(const slist& __x) : _Base(__x.get_allocator())
    { _M_insert_after_range(&this->_M_head, __x.begin(), __x.end()); }

  slist& operator= (const slist& __x);

  ~slist() {}
  
  // ...
};
```

迭代器与相关操作：

```c++
iterator begin() { return iterator((_Node*)this->_M_head._M_next); } // 头结点的下一个元素
const_iterator begin() const 
  { return const_iterator((_Node*)this->_M_head._M_next);}

iterator end() { return iterator(0); }                               // 空指针
const_iterator end() const { return const_iterator(0); }

// Experimental new feature: before_begin() returns a
// non-dereferenceable iterator that, when incremented, yields
// begin().  This iterator may be used as the argument to
// insert_after, erase_after, etc.  Note that even for an empty 
// slist, before_begin() is not the same iterator as end().  It 
// is always necessary to increment before_begin() at least once to
// obtain end().
iterator before_begin() { return iterator((_Node*) &this->_M_head); } // 头结点
const_iterator before_begin() const
  { return const_iterator((_Node*) &this->_M_head); }

size_type size() const { return __slist_size(this->_M_head._M_next); }

size_type max_size() const { return size_type(-1); }

bool empty() const { return this->_M_head._M_next == 0; }

void swap(slist& __x)
  { __STD::swap(this->_M_head._M_next, __x._M_head._M_next); } // 交换两个头结点的 next
```

支持从 slist 的头部进行操作：

```c++
reference front() { return ((_Node*) this->_M_head._M_next)->_M_data; }
const_reference front() const 
    { return ((_Node*) this->_M_head._M_next)->_M_data; }
void push_front(const value_type& __x)   {
    __slist_make_link(&this->_M_head, _M_create_node(__x));
}
void push_front() { __slist_make_link(&this->_M_head, _M_create_node()); }
void pop_front() {
    _Node* __node = (_Node*) this->_M_head._M_next;
    this->_M_head._M_next = __node->_M_next;
    destroy(&__node->_M_data);
    this->_M_put_node(__node);
}
```

返回迭代器的前一个位置 (需要从头遍历)：

```c++
iterator previous(const_iterator __pos) {
    return iterator((_Node*) __slist_previous(&this->_M_head, __pos._M_node));
}
const_iterator previous(const_iterator __pos) const {
    return const_iterator((_Node*) __slist_previous(&this->_M_head,
                                                    __pos._M_node));
}
```

效率较高的插入：`insert_after()`：

```c++
_Node* _M_insert_after(_Node_base* __pos, const value_type& __x) {
    return (_Node*) (__slist_make_link(__pos, _M_create_node(__x))); // 在当前位置之后插入
}

_Node* _M_insert_after(_Node_base* __pos) {
    return (_Node*) (__slist_make_link(__pos, _M_create_node()));    // 在当前位置之后插入
}

void _M_insert_after_fill(_Node_base* __pos,
                          size_type __n, const value_type& __x) {
    for (size_type __i = 0; __i < __n; ++__i)
        __pos = __slist_make_link(__pos, _M_create_node(__x));       // 在当前位置之后插入多个
}

void _M_insert_after_range(_Node_base* __pos,                        // 在当前位置之后插入一个范围
                           const_iterator __first, const_iterator __last) {
    while (__first != __last) {
        __pos = __slist_make_link(__pos, _M_create_node(*__first));
        ++__first;
    }
}
void _M_insert_after_range(_Node_base* __pos,                        // 在当前位置之后插入一个范围
                           const value_type* __first,
                           const value_type* __last) {
    while (__first != __last) {
        __pos = __slist_make_link(__pos, _M_create_node(*__first));
        ++__first;
    }
}
```

---

