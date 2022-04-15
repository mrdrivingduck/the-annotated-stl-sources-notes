# Chapter 4.5 - stack

Created by : Mr Dk.

2021 / 04 / 04 14:44 🍀

Nanjing, Jiangsu, China

---

## 4.5.1 stack 概述

stack 是一种 FILO (First In Last Out) 的数据结构，只有一个出口。允许以下操作：

- 新增元素
- 移除元素
- 取得最顶端元素

此外，没有其它方法可以存取 stack 中的元素，也没有遍历操作。

## 4.5.2 stack 定义完整列表

deque 是双向开口的数据结构。如果以 deque 为底层结构，封闭 deque 的头部开口，就形成了一个 stack。SGI STL 默认以 deque 为 stack 的底层结构。stack 的所有 API 都由底层数据结构的 API 来实现，只不过屏蔽了某些底层结构的 API (比如头部开口)。具有这种 _修改某物接口，形成另一种风貌_ 性质的结构被称为 **adapter (适配器)**。STL 中的 stack 通常不被归类为容器 (container)，而是容器适配器 (container adapter)。

```c++
template <class _Tp,
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(deque<_Tp>) > // deque
class stack;
```

```c++
template <class _Tp, class _Sequence>
class stack {

  // requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_REQUIRES(_Sequence, _BackInsertionSequence);
  typedef typename _Sequence::value_type _Sequence_value_type;
  __STL_CLASS_REQUIRES_SAME_TYPE(_Tp, _Sequence_value_type);


#ifdef __STL_MEMBER_TEMPLATES
  template <class _Tp1, class _Seq1>
  friend bool operator== (const stack<_Tp1, _Seq1>&,
                          const stack<_Tp1, _Seq1>&);
  template <class _Tp1, class _Seq1>
  friend bool operator< (const stack<_Tp1, _Seq1>&,
                         const stack<_Tp1, _Seq1>&);
#else /* __STL_MEMBER_TEMPLATES */
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const stack&, const stack&);
  friend bool __STD_QUALIFIER
  operator< __STL_NULL_TMPL_ARGS (const stack&, const stack&);
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
  stack() : c() {}
  explicit stack(const _Sequence& __s) : c(__s) {}

  // 以下 API 全部转而调用底层容器的 API 实现相应功能
  // 只暴露底层容器的尾部接口
  bool empty() const { return c.empty(); }
  size_type size() const { return c.size(); }
  reference top() { return c.back(); }
  const_reference top() const { return c.back(); }
  void push(const value_type& __x) { c.push_back(__x); }
  void pop() { c.pop_back(); }
};
```

运算符重载也直接调用底层容器的运算符：

```c++
template <class _Tp, class _Seq>
bool operator==(const stack<_Tp,_Seq>& __x, const stack<_Tp,_Seq>& __y)
{
  return __x.c == __y.c;
}

template <class _Tp, class _Seq>
bool operator<(const stack<_Tp,_Seq>& __x, const stack<_Tp,_Seq>& __y)
{
  return __x.c < __y.c;
}
```

## 4.5.3 stack 没有迭代器

stack 不提供遍历功能，也不提供迭代器。

## 4.5.4 以 list 作为 stack 的底层容器

list 也是双向开口的结构，并且 stack 使用的底层容器操作 `empty()` / `size()` / `back()` / `push_back()` / `pop_back()` list 也都有。因此也可以使用 list 作为 stack 的底层结构：

```c++
stack<int, list<int> > list_stack;
```
