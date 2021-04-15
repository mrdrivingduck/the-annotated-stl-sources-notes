# Chapter 6.7.2 - lower_bound

Created by : Mr Dk.

2021 / 04 / 14 22:39

Nanjing, Jiangsu, China

---

## lower_bound

二分查找的一种特殊版本，返回一个迭代器，指向第一个 **不小于指定值的元素**。换句话说，返回值是 **不破坏排序状态的原则下，可插入 value 的第一个位置**。算法默认使用 `operator<` 进行元素比较，用户也可以自定义二元仿函数。

```c++
template <class _ForwardIter, class _Tp, class _Distance>
_ForwardIter __lower_bound(_ForwardIter __first, _ForwardIter __last,
                           const _Tp& __val, _Distance*) 
{
  _Distance __len = 0;    // 标识区间长度
  distance(__first, __last, __len);
  _Distance __half;       // 标识区间长度的一半
  _ForwardIter __middle;  // 标识区间中点

  while (__len > 0) {
    __half = __len >> 1;
    __middle = __first;
    advance(__middle, __half);     // middle 指向区间中点
    if (*__middle < __val) {       // 区间中点 < 指定值，那么需要在区间中点之后的区间中寻找目标位置
      __first = __middle;
      ++__first;                   // 区间起点设置为当前中点的下一个位置
      __len = __len - __half - 1;  // 区间长度减半再减一
    }
    else                           // 否则指定值 <= 区间中点，需要在区间中点之前的区间中寻找目标位置
      __len = __half;              // 区间起点不变，区间长度减半
  }
  return __first;
}

template <class _ForwardIter, class _Tp>
inline _ForwardIter lower_bound(_ForwardIter __first, _ForwardIter __last,
				const _Tp& __val) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES_SAME_TYPE(_Tp,
      typename iterator_traits<_ForwardIter>::value_type);
  __STL_REQUIRES(_Tp, _LessThanComparable);
  return __lower_bound(__first, __last, __val,
                       __DISTANCE_TYPE(__first));
}

template <class _ForwardIter, class _Tp, class _Compare, class _Distance>
_ForwardIter __lower_bound(_ForwardIter __first, _ForwardIter __last,
                              const _Tp& __val, _Compare __comp, _Distance*);

template <class _ForwardIter, class _Tp, class _Compare>
inline _ForwardIter lower_bound(_ForwardIter __first, _ForwardIter __last,
                                const _Tp& __val, _Compare __comp);
```

## binary_search

找到有序区间中是否存在等于特定值的元素，如果有，则返回 `true`。这里借用了 `lower_bound()` 的实现，判断 `lower_bound()` 返回的迭代器指向的元素值是否等于特定值：

```c++
template <class _ForwardIter, class _Tp>
bool binary_search(_ForwardIter __first, _ForwardIter __last,
                   const _Tp& __val) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES_SAME_TYPE(_Tp,
        typename iterator_traits<_ForwardIter>::value_type);
  __STL_REQUIRES(_Tp, _LessThanComparable);
  _ForwardIter __i = lower_bound(__first, __last, __val);  // lower_bound 返回 val <= *__i 的元素
  return __i != __last && !(__val < *__i);                 // 排除 val < *i 的元素
}
```

---

