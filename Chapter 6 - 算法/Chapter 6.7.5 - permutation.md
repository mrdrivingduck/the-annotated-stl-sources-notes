# Chapter 6.7.5 - permutation

Created by : Mr Dk.

2021 / 04 / 15 16:27

Nanjing, Jiangsu, China

---

STL 提供了获取序列 _前一个_ / _后一个_ 排列组合序列的算法。所谓 _前一个 / 后一个_ 是指将序列中的所有元素根据 `operator<` 进行字典序排序后的 _前一个 / 后一个_ 序列。函数默认使用 `operator<` 来进行比较，但用户可以自行指定二元仿函数进行比较。

## next_permutation

从结尾开始寻找一对递增元素 `(a, b)`，然后再从结尾开始寻找第一个值大于 `a` 的元素 `c`。将 `a` 与 `c` 交换顺序后，将从 `b` 开始之后的区间进行颠倒。

> 为什么可以这样？

```cpp
// next_permutation and prev_permutation, with and without an explicitly
// supplied comparison function.

template <class _BidirectionalIter>
bool next_permutation(_BidirectionalIter __first, _BidirectionalIter __last) {
  __STL_REQUIRES(_BidirectionalIter, _BidirectionalIterator);
  __STL_REQUIRES(typename iterator_traits<_BidirectionalIter>::value_type,
                 _LessThanComparable);
  if (__first == __last)
    return false;
  _BidirectionalIter __i = __first;
  ++__i;
  if (__i == __last)
    return false;
  __i = __last;
  --__i;

  for(;;) {
    _BidirectionalIter __ii = __i;
    --__i;                          // i 与 ii 为一对相邻元素
    if (*__i < *__ii) {             // (i, ii) 递增
      _BidirectionalIter __j = __last;
      while (!(*__i < *--__j))      // 从结尾开始寻找第一个不小于 i 的元素 j
        {}
      iter_swap(__i, __j);          // i 与 j 互换
      reverse(__ii, __last);        // 将从 ii 开始之后的区间颠倒
      return true;
    }
    if (__i == __first) {           // (i, ii) 不递增，且 i 已经是第一个元素
      reverse(__first, __last);     // 颠倒整个区间
      return false;
    }
  }
}

template <class _BidirectionalIter, class _Compare>
bool next_permutation(_BidirectionalIter __first, _BidirectionalIter __last,
                      _Compare __comp);
```

## prev_permutation

与 `next_permutation()` 类似。从结尾开始寻找第一对递减的相邻元素 `(a, b)`，然后再从结尾开始寻找第一个不大于 `a` 的元素 `c`。将 `a` 与 `c` 对换后，将 `b` 开始到结尾的区间颠倒。

```cpp
template <class _BidirectionalIter>
bool prev_permutation(_BidirectionalIter __first, _BidirectionalIter __last) {
  __STL_REQUIRES(_BidirectionalIter, _BidirectionalIterator);
  __STL_REQUIRES(typename iterator_traits<_BidirectionalIter>::value_type,
                 _LessThanComparable);
  if (__first == __last)
    return false;
  _BidirectionalIter __i = __first;
  ++__i;
  if (__i == __last)
    return false;
  __i = __last;
  --__i;

  for(;;) {
    _BidirectionalIter __ii = __i;
    --__i;
    if (*__ii < *__i) {                 // 寻找到相邻的递减区间 (i, ii)
      _BidirectionalIter __j = __last;
      while (!(*--__j < *__i))          // 从结尾寻找第一个不小于 i 的元素 j
        {}
      iter_swap(__i, __j);              // i 与 j 互换
      reverse(__ii, __last);            // 将从 ii 开始的区间颠倒
      return true;
    }
    if (__i == __first) {
      reverse(__first, __last);
      return false;
    }
  }
}

template <class _BidirectionalIter, class _Compare>
bool prev_permutation(_BidirectionalIter __first, _BidirectionalIter __last,
                      _Compare __comp);
```
