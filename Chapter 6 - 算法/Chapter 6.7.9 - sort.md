# Chapter 6.7.9 - sort

Created by : Mr Dk.

2021 / 04 / 15 22:25

Nanjing, Jiangsu, China

---

STL 中的 `sort()` 接受两个 Random Access Iterator，默认使用 `operator<` 使元素从小到大排列，但也允许用户指定一个二元仿函数作为排序标准。

- STL 中的关系型容器由于本身就是有序的，因此并不需要用到这个算法
- 序列式容器中拥有特定出入口的容器 (stack / queue / priority_queue) 不能使用这个算法
- list 和 slist 由于提供的迭代器类型不属于随机访问迭代器，因此只能使用它们自己的成员函数 `sort()`

剩下只有 vector 和 deque 能够使用这个算法。

STL 的 `sort()` 在数据量较大时使用 **快速排序算法** 进行分段递归排序；当数据量低于某个阈值后，改用 **插入排序算法**；如果递归的层次过深，改用 **堆排序算法**。

## Insertion Sort

插入排序算法是一个经典的两层循环。将序列的前端视为有序区，后端视为无序区。每轮外层循环将无序区的第一个元素插入有序区中，而具体的插入位置需要通过内层循环来确定。内存循环通过不断交换 _逆序对 (inversion)_ 完成排序。这个算法的复杂度为 O(N^2)，但在数据量较少时有不错的效果；另外，在数据已经大致有序的前提下，性能也会很不错。

```cpp
template <class _RandomAccessIter>
void __insertion_sort(_RandomAccessIter __first, _RandomAccessIter __last) {
  if (__first == __last) return;
  for (_RandomAccessIter __i = __first + 1; __i != __last; ++__i)  // 从无序区的第一个元素开始循环
    __linear_insert(__first, __i, __VALUE_TYPE(__first));
}

template <class _RandomAccessIter, class _Tp>
inline void __linear_insert(_RandomAccessIter __first,
                            _RandomAccessIter __last, _Tp*) {
  _Tp __val = *__last;     // 待插入的元素
  if (__val < *__first) {  // 待插入的元素比第一个元素都小
    copy_backward(__first, __last, __last + 1);
    *__first = __val;      // 直接将整个有序区后移，并将待插入放到第一个位置上
  }  // 该分支使得下面的内存循环不需要判断循环区间是否到达边界，省下内层循环中的一个判断操作
  else
    __unguarded_linear_insert(__last, __val);  // 否则开始插入
}

template <class _RandomAccessIter, class _Tp>
void __unguarded_linear_insert(_RandomAccessIter __last, _Tp __val) {
  _RandomAccessIter __next = __last;
  --__next;
  while (__val < *__next) {  // 待插入元素小于当前元素 (目前是一个逆序对)
    *__last = *__next;       // 对换逆序对
    __last = __next;         // 探索前一个位置
    --__next;
  }
  *__last = __val;  // 将待插入元素放入最终的插入位置
}
```

## Quick Sort

快速排序是目前已知最快的排序方法，平均复杂度为 O(N log(N))，最坏情况将达到 O(N^2)。快速排序的核心在于将大区间分割为小区间，然后分段递归。最坏的情况发生在枢轴分割时产生了一个空的子区间，这样完全没有达到分割的预期效果。

### Median-of-Three

任何一个元素都可以被选为枢轴，但合适与否会影响快速排序的效率。为了避免序列元素中的非随机性，可以选择序列中头、中、尾三个位置的元素，以其中的中间值作为枢轴。为了能够快速取出序列中间位置的值，迭代器需要能够随机访问。

```cpp
template <class _Tp>
inline const _Tp& __median(const _Tp& __a, const _Tp& __b, const _Tp& __c) {
  __STL_REQUIRES(_Tp, _LessThanComparable);
  if (__a < __b)
    if (__b < __c)
      return __b;
    else if (__a < __c)
      return __c;
    else
      return __a;
  else if (__a < __c)
    return __a;
  else if (__b < __c)
    return __c;
  else
    return __b;
}

template <class _Tp, class _Compare>
inline const _Tp&
__median(const _Tp& __a, const _Tp& __b, const _Tp& __c, _Compare __comp);
```

### Partitioning

使头部迭代器和尾部迭代器同时向中间移动，知道不满足枢轴条件时停下。如果两个迭代器没有交错，那么元素互换；如果两个迭代器已经交错，说明整个序列已经分割完毕。

```cpp
template <class _RandomAccessIter, class _Tp>
_RandomAccessIter __unguarded_partition(_RandomAccessIter __first,
                                        _RandomAccessIter __last,
                                        _Tp __pivot)
{
  while (true) {
    while (*__first < __pivot)  // 从前寻找第一个不满足枢轴条件的元素
      ++__first;
    --__last;
    while (__pivot < *__last)   // 从后寻找第一个不满足枢轴条件的元素
      --__last;
    if (!(__first < __last))    // 前后迭代器已经交错
      return __first;           // 返回
    iter_swap(__first, __last);  // 交换不满足枢轴条件的两端元素
    ++__first;                   // 继续检查下一个元素
  }
}

template <class _RandomAccessIter, class _Tp, class _Compare>
_RandomAccessIter __unguarded_partition(_RandomAccessIter __first,
                                        _RandomAccessIter __last,
                                        _Tp __pivot, _Compare __comp);
```

### threshold

插入排序和快速排序的切换阈值为多少？实际最佳值因设备而异。

### final insertion sort

插入排序算法在面对 _几乎已经有序_ 的序列时，有着很好的表现。

### introsort

由 _David R.Musser_ 提出，当分割行为有向 O(N^2) 转化的倾向时，自动侦测并转而改用堆排序。

### SGI STL sort

```cpp
template <class _RandomAccessIter>
inline void sort(_RandomAccessIter __first, _RandomAccessIter __last) {
  __STL_REQUIRES(_RandomAccessIter, _Mutable_RandomAccessIterator);
  __STL_REQUIRES(typename iterator_traits<_RandomAccessIter>::value_type,
                 _LessThanComparable);
  if (__first != __last) {
    __introsort_loop(__first, __last,              // 进行一轮 introsort
                     __VALUE_TYPE(__first),
                     __lg(__last - __first) * 2);  // 递归深度计算
    __final_insertion_sort(__first, __last);       // 最终进行一次插入排序
  }
}

template <class _Size>
inline _Size __lg(_Size __n) {
  _Size __k;
  for (__k = 0; __n != 1; __n >>= 1) ++__k;
  return __k;
}
```

上述排序包含两个过程：

1. introsort
2. final_insertion_sort

```cpp
template <class _RandomAccessIter, class _Tp, class _Size>
void __introsort_loop(_RandomAccessIter __first,
                      _RandomAccessIter __last, _Tp*,
                      _Size __depth_limit)
{
  while (__last - __first > __stl_threshold) {  // 元素数量大于阈值，使用快速排序
    if (__depth_limit == 0) {                   // 不再允许加深递归
      partial_sort(__first, __last, __last);    // 改为使用堆排序
      return;
    }
    --__depth_limit;  // 递归深度减 1，准备递归
    _RandomAccessIter __cut =
      __unguarded_partition(__first, __last,
                            _Tp(__median(*__first,  // 选择合适的枢轴，并进行切分
                                         *(__first + (__last - __first)/2),
                                         *(__last - 1))));
    __introsort_loop(__cut, __last, (_Tp*) 0, __depth_limit);  // 对后半区间递归排序
    __last = __cut;  // 将区间缩小为前半区间，对前半区间进行排序
  }
}

const int __stl_threshold = 16;  // 开始 introsort 的阈值个数为 16
```

```cpp
template <class _RandomAccessIter>
void __final_insertion_sort(_RandomAccessIter __first,
                            _RandomAccessIter __last) {
  if (__last - __first > __stl_threshold) {  // 区间内元素个数多于阈值
    __insertion_sort(__first, __first + __stl_threshold);  // 对阈值区间直接进行插入排序 (这部分没有被 introsort 排序)
    __unguarded_insertion_sort(__first + __stl_threshold, __last);  // 阈值区间以外的元素进行插入排序
  }
  else                                  // 区间内元素个数少于阈值
    __insertion_sort(__first, __last);  // 直接进行插入排序
}

template <class _RandomAccessIter>
inline void __unguarded_insertion_sort(_RandomAccessIter __first,
                                _RandomAccessIter __last) {
  __unguarded_insertion_sort_aux(__first, __last, __VALUE_TYPE(__first));
}

template <class _RandomAccessIter, class _Tp>
void __unguarded_insertion_sort_aux(_RandomAccessIter __first,
                                    _RandomAccessIter __last, _Tp*) {
  for (_RandomAccessIter __i = __first; __i != __last; ++__i)
    __unguarded_linear_insert(__i, _Tp(*__i));  // 双层循环的插入排序
}
```
