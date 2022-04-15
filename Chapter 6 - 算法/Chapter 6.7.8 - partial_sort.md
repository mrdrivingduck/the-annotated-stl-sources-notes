# Chapter 6.7.8 - partial_sort / partial_sort_copy

Created by : Mr Dk.

2021 / 04 / 15 16:57

Nanjing, Jiangsu, China

---

算法接收一个处于 `[first, last)` 之间的迭代器 `middle`，使得序列中 `middle - first` 个最小元素按递增顺序排序，放置于 `[first, middle)` 内；其余 `last - middle` 个元素放置在 `[middle, last)` 中，不保证任何顺序。顾名思义，只对序列的前 `middle - first` 个最小元素进行部分排序。

算法根据 `operator<` 或用户自定义的二元仿函数来决定如何比较元素。

算法使用 **堆排序** 实现 (与我预想的部分快速排序的方式不同)。首先将 `[first, middle)` 区间内的所有元素通过 `make_heap()` 维护为一个大顶堆，然后将 `[middle, last)` 区间内的每个元素依次与堆顶比较，堆顶元素为目前 `[first, middle)` 区间内的最大值：如果比堆顶元素小，那么将堆顶元素与当前元素替换，然后重整堆。最终，堆内应当维护的是整个序列中最小的 `middle - first` 个数，对这个区间进行堆排序 `sort_heap()` 即可。

## partial_sort

```c++
// partial_sort, partial_sort_copy, and auxiliary functions.

template <class _RandomAccessIter>
inline void partial_sort(_RandomAccessIter __first,
                         _RandomAccessIter __middle,
                         _RandomAccessIter __last) {
  __STL_REQUIRES(_RandomAccessIter, _Mutable_RandomAccessIterator);
  __STL_REQUIRES(typename iterator_traits<_RandomAccessIter>::value_type,
                 _LessThanComparable);
  __partial_sort(__first, __middle, __last, __VALUE_TYPE(__first));
}

template <class _RandomAccessIter, class _Tp>
void __partial_sort(_RandomAccessIter __first, _RandomAccessIter __middle,
                    _RandomAccessIter __last, _Tp*) {
  make_heap(__first, __middle);  // 对前 middle - first 个元素建堆
  for (_RandomAccessIter __i = __middle; __i < __last; ++__i)  // 遍历之后区间的每个元素
    if (*__i < *__first)                                       // 元素小于堆顶
      __pop_heap(__first, __middle, __i, _Tp(*__i),            // 堆顶元素从堆中移除，并换成当前遍历元素
                 __DISTANCE_TYPE(__first));
  sort_heap(__first, __middle);  // 对前 middle - first 个元素进行堆排序
}

template <class _RandomAccessIter, class _Compare>
inline void partial_sort(_RandomAccessIter __first,
                         _RandomAccessIter __middle,
                         _RandomAccessIter __last, _Compare __comp);
```

## partial_sort_copy

先将区间复制到另一块空间中，然后在复制后的空间中完成部分排序。

```c++
template <class _InputIter, class _RandomAccessIter>
inline _RandomAccessIter
partial_sort_copy(_InputIter __first, _InputIter __last,
                  _RandomAccessIter __result_first,
                  _RandomAccessIter __result_last) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_RandomAccessIter, _Mutable_RandomAccessIterator);
  __STL_CONVERTIBLE(typename iterator_traits<_InputIter>::value_type,
                    typename iterator_traits<_RandomAccessIter>::value_type);
  __STL_REQUIRES(typename iterator_traits<_RandomAccessIter>::value_type,
                 _LessThanComparable);
  __STL_REQUIRES(typename iterator_traits<_InputIter>::value_type,
                 _LessThanComparable);
  return __partial_sort_copy(__first, __last, __result_first, __result_last,
                             __DISTANCE_TYPE(__result_first),
                             __VALUE_TYPE(__first));
}

template <class _InputIter, class _RandomAccessIter, class _Distance,
          class _Tp>
_RandomAccessIter __partial_sort_copy(_InputIter __first,
                                      _InputIter __last,
                                      _RandomAccessIter __result_first,
                                      _RandomAccessIter __result_last,
                                      _Distance*, _Tp*) {
  if (__result_first == __result_last) return __result_last;
  _RandomAccessIter __result_real_last = __result_first;
  while(__first != __last && __result_real_last != __result_last) {  // 复制所有元素到新空间中
    *__result_real_last = *__first;
    ++__result_real_last;
    ++__first;
  }
  make_heap(__result_first, __result_real_last);   // 对整个复制后的区间建堆？
  while (__first != __last) {
    if (*__first < *__result_first)                // 当前遍历元素小于目前堆顶
      __adjust_heap(__result_first, _Distance(0),  // 当前遍历元素赋值给堆顶，并调整堆序
                    _Distance(__result_real_last - __result_first),
                    _Tp(*__first));
    ++__first;  // 遍历下一个元素
  }
  sort_heap(__result_first, __result_real_last);   // 对部分排序区间进行堆排序
  return __result_real_last;
}
```
