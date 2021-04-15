# Chapter 6.7.3 - upper_bound

Created by : Mr Dk.

2021 / 04 / 15 10:37

Nanjing, Jiangsu, China

---

二分查找的一个特殊版本。在不破坏顺序的前提下，寻找 **可插入特定值的最后一个合适的位置**。默认使用 `operator<` 进行比较，但也支持用户提供的二元仿函数。

```c++
template <class _ForwardIter, class _Tp, class _Distance>
_ForwardIter __upper_bound(_ForwardIter __first, _ForwardIter __last,
                           const _Tp& __val, _Distance*)
{
  _Distance __len = 0;
  distance(__first, __last, __len);
  _Distance __half;
  _ForwardIter __middle;

  while (__len > 0) {
    __half = __len >> 1;        // 区间长度减半
    __middle = __first;
    advance(__middle, __half);  // middle 指向区间中点
    if (__val < *__middle)      // 指定值 < 区间中点值，那么要到前半区间寻找
      __len = __half;           // 区间长度减半
    else {                      // 指定值 >= 区间中点值
      __first = __middle;          // 在后半区间寻找
      ++__first;                   // first 指向区间中点的下一个值 (因为 STL 的插入位置在目标迭代器之前)
      __len = __len - __half - 1;  // 区间长度减半再减 1
    }
  }
  return __first;
}

template <class _ForwardIter, class _Tp>
inline _ForwardIter upper_bound(_ForwardIter __first, _ForwardIter __last,
                                const _Tp& __val) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES_SAME_TYPE(_Tp,
      typename iterator_traits<_ForwardIter>::value_type);
  __STL_REQUIRES(_Tp, _LessThanComparable);
  return __upper_bound(__first, __last, __val,
                       __DISTANCE_TYPE(__first));
}

template <class _ForwardIter, class _Tp, class _Compare>
inline _ForwardIter upper_bound(_ForwardIter __first, _ForwardIter __last,
                                const _Tp& __val, _Compare __comp);  // 二元仿函数版本
```

> 与 `lower_bound()` 的差别是使用 `operator<` (或用户提供的仿函数) 时操作数的顺序不一样。
>
> `lower_bound()`：
>
> ```c++
> if (*__middle < __val) { /* ... */ }
> ```
>
> `upper_bound()`：
>
> ```c++
> if (__val < *__middle) { /* ... */ }
> ```
>
> 带来的差异就是，`lower_bound()` 寻找特定值的第一个插入位置，`upper_bound()` 寻找特定值的最后一个插入位置。

---

