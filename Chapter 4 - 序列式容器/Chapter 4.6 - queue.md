# Chapter 4.6 - queue

Created by : Mr Dk.

2021 / 04 / 04 14:50 🍀

Nanjing, Jiangsu, China

---

## 4.6.1 queue 概述

queue 是一种 FIFO (First In First Out) 的数据结构。允许从尾端加入元素，从头部取出元素。此外没有方法可以访问 queue 中的元素。

## 4.6.2 queue 定义完整列表

deque 是双向开口结构，如果封闭其头部的入口，以及尾部的出口，那么就很容易形成一个 queue。SGI STL 默认以 deque 作为 queue 的底层结构。同样，queue 借用了底层结构的 API，并屏蔽了其中的一些 API，因此也是一个 container adapter。

```c++
template <class _Tp,
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(deque<_Tp>) >
class queue;
```

```c++
template <class _Tp, class _Sequence>
class queue {

  // requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_REQUIRES(_Sequence, _FrontInsertionSequence);
  __STL_CLASS_REQUIRES(_Sequence, _BackInsertionSequence);
  typedef typename _Sequence::value_type _Sequence_value_type;
  __STL_CLASS_REQUIRES_SAME_TYPE(_Tp, _Sequence_value_type);


#ifdef __STL_MEMBER_TEMPLATES
  template <class _Tp1, class _Seq1>
  friend bool operator== (const queue<_Tp1, _Seq1>&,
                          const queue<_Tp1, _Seq1>&);
  template <class _Tp1, class _Seq1>
  friend bool operator< (const queue<_Tp1, _Seq1>&,
                         const queue<_Tp1, _Seq1>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const queue&, const queue&);
  friend bool __STD_QUALIFIER
  operator<  __STL_NULL_TMPL_ARGS (const queue&, const queue&);
#endif /* __STL_MEMBER_TEMPLATES */

public:
  typedef typename _Sequence::value_type      value_type;
  typedef typename _Sequence::size_type       size_type;
  typedef          _Sequence                  container_type;

  typedef typename _Sequence::reference       reference;
  typedef typename _Sequence::const_reference const_reference;
protected:
  _Sequence c; // 底层容器
public:
  queue() : c() {}
  explicit queue(const _Sequence& __c) : c(__c) {}

  bool empty() const { return c.empty(); }
  size_type size() const { return c.size(); }
  reference front() { return c.front(); }
  const_reference front() const { return c.front(); }
  reference back() { return c.back(); }
  const_reference back() const { return c.back(); }
  void push(const value_type& __x) { c.push_back(__x); } // 只能从尾部 push
  void pop() { c.pop_front(); }                          // 只能从头部 pop
};
```

运算符重载也是借用底层容器的运算符：

```c++
template <class _Tp, class _Sequence>
bool
operator==(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return __x.c == __y.c;
}

template <class _Tp, class _Sequence>
bool
operator<(const queue<_Tp, _Sequence>& __x, const queue<_Tp, _Sequence>& __y)
{
  return __x.c < __y.c;
}
```

## 4.6.3 queue 没有迭代器

queue 不提供遍历功能，也不提供迭代器。

## 4.6.4 以 list 作为 queue 的底层容器

除了 deque，list 也是双向开口的数据结构，同样可以适配到 queue 上：

```c++
queue<int, list<int> > list_queue;
```
