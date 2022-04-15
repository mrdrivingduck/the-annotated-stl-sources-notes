# Chapter 6.7.12 - nth_element

Created by : Mr Dk.

2021 / 04 / 15 22:54

Nanjing, Jiangsu, China

---

对区间进行重新排列，并保证以指定位置的元素为枢轴，该位置之前的元素都不大于该元素，该位置之后的元素都不小于该元素，而前后两个区间内的元素次序则不做保证。默认以 `operator<` 进行比较，也可以接收用户提供的二元仿函数。

算法不断以 median-of-3 partitioning 分割法进行切分，并判断分割完毕的枢轴位置与指定的枢轴位置的相对差。如果指定枢轴位于分割完毕的枢轴左边，那么对左区间进行进一步分割；否则对右区间进行进一步分割。当分割元素过少时 (小于等于 3)，那么分割操作实际上等效于三个元素的排序 (小于枢轴 / 枢轴 / 大于枢轴)，所以可以直接用插入排序实现。

```c++
template <class _RandomAccessIter>
inline void nth_element(_RandomAccessIter __first, _RandomAccessIter __nth,
                        _RandomAccessIter __last) {
  __STL_REQUIRES(_RandomAccessIter, _Mutable_RandomAccessIterator);
  __STL_REQUIRES(typename iterator_traits<_RandomAccessIter>::value_type,
                 _LessThanComparable);
  __nth_element(__first, __nth, __last, __VALUE_TYPE(__first));
}

template <class _RandomAccessIter, class _Tp, class _Compare>
void __nth_element(_RandomAccessIter __first, _RandomAccessIter __nth,
                   _RandomAccessIter __last, _Tp*, _Compare __comp);
```

```c++
template <class _RandomAccessIter, class _Tp>
void __nth_element(_RandomAccessIter __first, _RandomAccessIter __nth,
                   _RandomAccessIter __last, _Tp*) {
  while (__last - __first > 3) {  // 区间内元素超过 3 个
    _RandomAccessIter __cut =
      __unguarded_partition(__first, __last,
                            _Tp(__median(*__first,  // 选择合适的枢轴，分割
                                         *(__first + (__last - __first)/2),
                                         *(__last - 1))));
    if (__cut <= __nth)  // 目标位置位于本轮分割后的右区间中
      __first = __cut;   // 进一步分割右区间
    else                 // 目标位置位于本轮分割后的左区间中
      __last = __cut;    // 进一步分割左区间
  }
  __insertion_sort(__first, __last);  // 区间内元素不超过 3 个，则直接使用快速排序
}
```
