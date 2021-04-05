# Chapter 4.8 - priority_queue

Created by : Mr Dk.

2021 / 04 / 05 10:38

Nanjing, Jiangsu, China

---

## 4.8.1 priority_queue 概述

拥有权重观念的 queue。元素以任意次序被推入队列，以权重的高低顺序被取出。缺省情况下，priority_queue 利用一个 max-heap 实现，以 vector 为底层容器。因此 priority_queue 也是一个 container adapter。

## 4.8.2 priority_queue 完整定义列表

```c++
template <class _Tp, 
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(vector<_Tp>), // 底层容器
          class _Compare                                             // 比较运算算子
          __STL_DEPENDENT_DEFAULT_TMPL(less<typename _Sequence::value_type>) >
class priority_queue {

  // requirements:

  __STL_CLASS_REQUIRES(_Tp, _Assignable);
  __STL_CLASS_REQUIRES(_Sequence, _Sequence);
  __STL_CLASS_REQUIRES(_Sequence, _RandomAccessContainer);
  typedef typename _Sequence::value_type _Sequence_value_type;
  __STL_CLASS_REQUIRES_SAME_TYPE(_Tp, _Sequence_value_type);
  __STL_CLASS_BINARY_FUNCTION_CHECK(_Compare, bool, _Tp, _Tp);

public:
  typedef typename _Sequence::value_type      value_type;
  typedef typename _Sequence::size_type       size_type;
  typedef          _Sequence                  container_type;

  typedef typename _Sequence::reference       reference;
  typedef typename _Sequence::const_reference const_reference;
protected:
  _Sequence c;
  _Compare comp;
public:
  // 构造函数主要对 make_heap() 函数进行了实用
  priority_queue() : c() {}
  explicit priority_queue(const _Compare& __x) :  c(), comp(__x) {}
  priority_queue(const _Compare& __x, const _Sequence& __s) 
    : c(__s), comp(__x) 
    { make_heap(c.begin(), c.end(), comp); } 

      priority_queue(const value_type* __first, const value_type* __last) 
    : c(__first, __last) { make_heap(c.begin(), c.end(), comp); }

  priority_queue(const value_type* __first, const value_type* __last, 
                 const _Compare& __x) 
    : c(__first, __last), comp(__x)
    { make_heap(c.begin(), c.end(), comp); }

  priority_queue(const value_type* __first, const value_type* __last, 
                 const _Compare& __x, const _Sequence& __c)
    : c(__c), comp(__x) 
  { 
    c.insert(c.end(), __first, __last);
    make_heap(c.begin(), c.end(), comp);
  }
    
  // 容量操作使用了底层容器的 API
  bool empty() const { return c.empty(); }
  size_type size() const { return c.size(); }
  const_reference top() const { return c.front(); }
    
  // push 函数应用了 push_heap() 算法
  void push(const value_type& __x) {
    __STL_TRY {
      c.push_back(__x); 
      push_heap(c.begin(), c.end(), comp);
    }
    __STL_UNWIND(c.clear());
  }
  // pop 函数应用了 pop_heap() 算法
  void pop() {
    __STL_TRY {
      pop_heap(c.begin(), c.end(), comp);
      c.pop_back();
    }
    __STL_UNWIND(c.clear());
  }
};
```

## 4.8.3 priority_queue 没有迭代器

不提供遍历功能，不提供迭代器。

## 4.8.4 priority_queue 测试实例

```c++
priority_queue<int> q1;
priority_queue<int, deque<int>, greater<int>> q2; // 底层容器需要支持随机访问迭代器
```

---

