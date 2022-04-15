# Chapter 6.4 - 基本算法

Created by : Mr Dk.

2021 / 04 / 11 15:00

Nanjing, Jiangsu, China

---

## equal

比较两个区间内的对应元素是否相等。如果第二个序列的元素比较多，则多出来的元素不考虑 (看源码就知道为啥了)：因此如果想比较两个序列是否完全相等，需要先比较序列长度是否相等，或者直接使用容器提供的 `operator==`。

算法默认使用 `operator==` 对元素进行大小比较，用户也可以自行指定二元仿函数。

```c++
template <class _InputIter1, class _InputIter2>
inline bool equal(_InputIter1 __first1, _InputIter1 __last1,
                  _InputIter2 __first2) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _EqualityComparable);
  __STL_REQUIRES(typename iterator_traits<_InputIter2>::value_type,
                 _EqualityComparable);
  for ( ; __first1 != __last1; ++__first1, ++__first2)
    if (*__first1 != *__first2)  // 使用 == 运算符
      return false;  // 如果发现不相等，立刻返回
  return true;  // 第一序列内的所有元素都和第二序列对应相等
}

template <class _InputIter1, class _InputIter2, class _BinaryPredicate>
inline bool equal(_InputIter1 __first1, _InputIter1 __last1,
                  _InputIter2 __first2, _BinaryPredicate __binary_pred) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  for ( ; __first1 != __last1; ++__first1, ++__first2)
    if (!__binary_pred(*__first1, *__first2))  // 使用二元仿函数
      return false;
  return true;
}
```

## fill

将指定区间内的所有元素设置为新值。该函数只对元素进行赋值，不对元素进行构造，因此操作区间不能超过容器的实际大小。

```c++
template <class _ForwardIter, class _Tp>
void fill(_ForwardIter __first, _ForwardIter __last, const _Tp& __value) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  for ( ; __first != __last; ++__first)
    *__first = __value;
}
```

对于单字节的类型，直接使用 `memset()`。以下是部分具体化的版本：

```c++
inline void fill(unsigned char* __first, unsigned char* __last,
                 const unsigned char& __c) {
  unsigned char __tmp = __c;
  memset(__first, __tmp, __last - __first);
}

inline void fill(signed char* __first, signed char* __last,
                 const signed char& __c) {
  signed char __tmp = __c;
  memset(__first, static_cast<unsigned char>(__tmp), __last - __first);
}

inline void fill(char* __first, char* __last, const char& __c) {
  char __tmp = __c;
  memset(__first, static_cast<unsigned char>(__tmp), __last - __first);
}
```

将从某个位置开始的前 n 个元素设置为新值：

```c++
template <class _OutputIter, class _Size, class _Tp>
_OutputIter fill_n(_OutputIter __first, _Size __n, const _Tp& __value) {
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  for ( ; __n > 0; --__n, ++__first)  // 循环条件里多了 n
    *__first = __value;
  return __first;
}
```

如果想要获得的是插入行为而不是赋值，那么可以使用具有插入能力的迭代器适配器：

```c++
vector<int> iv;
fill_n(inserter(iv, iv.begin()), 5, 7);  // 在 iv.begin() 前插入 5 个 7
```

## iter_swap

将两个迭代器所指向的对象进行交换。由于交换操作需要使用到一个临时变量，如果不知道这个临时变量的数据类型，代码应该怎么写呢？如何声明这个临时变量呢？这里使用到了迭代器的数据类型萃取功能，提取出了迭代器的 value type：

```c++
template <class _ForwardIter1, class _ForwardIter2, class _Tp>
inline void __iter_swap(_ForwardIter1 __a, _ForwardIter2 __b, _Tp*) {  // _Tp 为迭代器所指对象类型
  _Tp __tmp = *__a;
  *__a = *__b;
  *__b = __tmp;
}

template <class _ForwardIter1, class _ForwardIter2>
inline void iter_swap(_ForwardIter1 __a, _ForwardIter2 __b) {
  __STL_REQUIRES(_ForwardIter1, _Mutable_ForwardIterator);
  __STL_REQUIRES(_ForwardIter2, _Mutable_ForwardIterator);
  __STL_CONVERTIBLE(typename iterator_traits<_ForwardIter1>::value_type,
                    typename iterator_traits<_ForwardIter2>::value_type);
  __STL_CONVERTIBLE(typename iterator_traits<_ForwardIter2>::value_type,
                    typename iterator_traits<_ForwardIter1>::value_type);
  __iter_swap(__a, __b, __VALUE_TYPE(__a));  // 萃取迭代器所指对象类型
}
```

## lexicographical_compare

按 **字典序** 对两个序列进行比较，直到两个序列的某组对应元素不相等，或者到达结尾。默认使用 `operator<` 进行比较，也支持用户提供二元仿函数。

```c++
//--------------------------------------------------
// lexicographical_compare and lexicographical_compare_3way.
// (the latter is not part of the C++ standard.)

template <class _InputIter1, class _InputIter2>
bool lexicographical_compare(_InputIter1 __first1, _InputIter1 __last1,
                             _InputIter2 __first2, _InputIter2 __last2) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _LessThanComparable);
  __STL_REQUIRES(typename iterator_traits<_InputIter2>::value_type,
                 _LessThanComparable);
  for ( ; __first1 != __last1 && __first2 != __last2
        ; ++__first1, ++__first2) {
    if (*__first1 < *__first2)  // 不相等，直接返回
      return true;
    if (*__first2 < *__first1)  // 不相等，直接返回
      return false;
  }
  // 之前的元素全部对应相等
  // 第一个序列到结尾，第二个序列还没有到结尾，则 1 < 2
  return __first1 == __last1 && __first2 != __last2;
}

template <class _InputIter1, class _InputIter2, class _Compare>
bool lexicographical_compare(_InputIter1 __first1, _InputIter1 __last1,
                             _InputIter2 __first2, _InputIter2 __last2,
                             _Compare __comp) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  for ( ; __first1 != __last1 && __first2 != __last2
        ; ++__first1, ++__first2) {
    if (__comp(*__first1, *__first2))  // 二元仿函数比较
      return true;
    if (__comp(*__first2, *__first1))  // 二元仿函数比较
      return false;
  }
  return __first1 == __last1 && __first2 != __last2;
}
```

部分具体化：对于单字节的字符串，使用 `memcmp()` 进行比较。如果已经比较完毕的部分大小不相上下，那么长度更长的字符串更大。

```c++
inline bool
lexicographical_compare(const unsigned char* __first1,
                        const unsigned char* __last1,
                        const unsigned char* __first2,
                        const unsigned char* __last2)
{
  const size_t __len1 = __last1 - __first1;
  const size_t __len2 = __last2 - __first2;
  const int __result = memcmp(__first1, __first2, min(__len1, __len2));  // 比较长度相同的部分
  return __result != 0 ? __result < 0 : __len1 < __len2;  // 结果为 0，则比较长度
}

inline bool lexicographical_compare(const char* __first1, const char* __last1,
                                    const char* __first2, const char* __last2)
{
  return lexicographical_compare((const signed char*) __first1,
                                 (const signed char*) __last1,
                                 (const signed char*) __first2,
                                 (const signed char*) __last2);
}
```

## max / min

默认使用 `operator<` 来获得两个对象中的最大 / 最小值，用户可以提供二元仿函数。

```c++
//--------------------------------------------------
// min and max

#if !defined(__BORLANDC__) || __BORLANDC__ >= 0x540 /* C++ Builder 4.0 */

#undef min
#undef max

template <class _Tp>
inline const _Tp& min(const _Tp& __a, const _Tp& __b) {
  __STL_REQUIRES(_Tp, _LessThanComparable);
  return __b < __a ? __b : __a;
}

template <class _Tp>
inline const _Tp& max(const _Tp& __a, const _Tp& __b) {
  __STL_REQUIRES(_Tp, _LessThanComparable);
  return  __a < __b ? __b : __a;
}

#endif /* __BORLANDC__ */

template <class _Tp, class _Compare>
inline const _Tp& min(const _Tp& __a, const _Tp& __b, _Compare __comp) {
  return __comp(__b, __a) ? __b : __a;  // 二元仿函数
}

template <class _Tp, class _Compare>
inline const _Tp& max(const _Tp& __a, const _Tp& __b, _Compare __comp) {
  return __comp(__a, __b) ? __b : __a;  // 二元仿函数
}
```

## mismatch

平行比较两个序列，并返回两个序列之间的第一个不匹配点 (迭代器)。如果第二个序列的元素个数更多，那么多出来的元素忽略不计。缺省情况使用 `operator==` 来比较元素，用户可以自行提供二元仿函数。

```c++
//--------------------------------------------------
// equal and mismatch

template <class _InputIter1, class _InputIter2>
pair<_InputIter1, _InputIter2> mismatch(_InputIter1 __first1,
                                        _InputIter1 __last1,
                                        _InputIter2 __first2) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _EqualityComparable);
  __STL_REQUIRES(typename iterator_traits<_InputIter2>::value_type,
                 _EqualityComparable);
  while (__first1 != __last1 && *__first1 == *__first2) {  // 第一序列没到尾，使用 operator== 比较
    ++__first1;
    ++__first2;
  }
  return pair<_InputIter1, _InputIter2>(__first1, __first2);  // 返回失配位置的迭代器
}

template <class _InputIter1, class _InputIter2, class _BinaryPredicate>
pair<_InputIter1, _InputIter2> mismatch(_InputIter1 __first1,
                                        _InputIter1 __last1,
                                        _InputIter2 __first2,
                                        _BinaryPredicate __binary_pred) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  while (__first1 != __last1 && __binary_pred(*__first1, *__first2)) {  // 第一序列没到尾，使用二元仿函数比较
    ++__first1;
    ++__first2;
  }
  return pair<_InputIter1, _InputIter2>(__first1, __first2);  // 返回失配位置的迭代器
}
```

## swap

对换两个对象中的内容 (使用 `operator=`)：

```c++
template <class _Tp>
inline void swap(_Tp& __a, _Tp& __b) {
  __STL_REQUIRES(_Tp, _Assignable);
  _Tp __tmp = __a;
  __a = __b;
  __b = __tmp;
}
```

## copy

在区间之间复制。由于复制基本上使用的是 `operator=` 或拷贝构造函数，对于拥有简单拷贝构造函数的对象来说，直接使用底层的内存复制能够节省大量的时间。SGI STL 中使用的是 `memmove()`：它会将整个区间复制下来，然后搬运。

`copy()` 将 `[first, last)` 区间内的元素复制到 `result` 开始的区间内。`first` 和 `result` 可以在同一个容器中，但是 `result` 如果处于 `first` 和 `last` 之间将会出现问题：因为 `result` 中的原有数据也是复制区间的一部分，而它将会被 `first` 的元素覆盖。换句话说，如果复制区间存在重叠，`copy()` **仅支持向前复制** (`result` 位置位于 `first` 之间)。

> 但是对于使用了 `memmove()` 的类型来说，不存在上述问题。

`copy()` 对性能上的优化分为两个层次：

- 针对参数给定的迭代器类型，可以使用不同的操作算法
- 针对被复制元素的数据类型，选择是否使用高效的内存移动算法

```c++
//--------------------------------------------------
// copy

// All of these auxiliary functions serve two purposes.  (1) Replace
// calls to copy with memmove whenever possible.  (Memmove, not memcpy,
// because the input and output ranges are permitted to overlap.)
// (2) If we're using random access iterators, then write the loop as
// a for loop with an explicit count.
```

最泛化的版本：接收两个 Input Iterator (被复制区间) 和一个 Output Iterator (目标区间起始位置) 进行复制。

```c++
template <class _InputIter, class _OutputIter>
inline _OutputIter copy(_InputIter __first, _InputIter __last,
                        _OutputIter __result)
{
  return __copy(__first, __last, __result,
                __ITERATOR_CATEGORY(__first),  // 萃取迭代器的类型
                __DISTANCE_TYPE(__first));
}
```

对于 Input Iterator，使用迭代器的 `operator==` 和 `operator++` 能够实现遍历；而对于 Ramdom Access Iterator 来说，可以直接计算两个迭代器的距离，并基于首尾迭代器的距离进行计算，效率更高：

```c++
template <class _InputIter, class _OutputIter, class _Distance>
inline _OutputIter __copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result,
                          input_iterator_tag, _Distance*)  // input iterator
{
  for ( ; __first != __last; ++__result, ++__first)
    *__result = *__first;
  return __result;
}

template <class _RandomAccessIter, class _OutputIter, class _Distance>
inline _OutputIter
__copy(_RandomAccessIter __first, _RandomAccessIter __last,
       _OutputIter __result, random_access_iterator_tag, _Distance*)  // random access iterator
{
  for (_Distance __n = __last - __first; __n > 0; --__n) {  // 计算首尾迭代器之间的距离
    *__result = *__first;
    ++__first;
    ++__result;
  }
  return __result;
}
```

而对于原生数据类型，SGI STL 认为它们具有简单的拷贝构造函数，因此直接可以使用 `memmove`：

```c++
#define __SGI_STL_DECLARE_COPY_TRIVIAL(_Tp)                                \
  inline _Tp* copy(const _Tp* __first, const _Tp* __last, _Tp* __result) { \
    memmove(__result, __first, sizeof(_Tp) * (__last - __first));          \
    return __result + (__last - __first);                                  \
  }

__SGI_STL_DECLARE_COPY_TRIVIAL(char)
__SGI_STL_DECLARE_COPY_TRIVIAL(signed char)
__SGI_STL_DECLARE_COPY_TRIVIAL(unsigned char)
__SGI_STL_DECLARE_COPY_TRIVIAL(short)
__SGI_STL_DECLARE_COPY_TRIVIAL(unsigned short)
__SGI_STL_DECLARE_COPY_TRIVIAL(int)
__SGI_STL_DECLARE_COPY_TRIVIAL(unsigned int)
__SGI_STL_DECLARE_COPY_TRIVIAL(long)
__SGI_STL_DECLARE_COPY_TRIVIAL(unsigned long)
#ifdef __STL_HAS_WCHAR_T
__SGI_STL_DECLARE_COPY_TRIVIAL(wchar_t)
#endif
#ifdef _STL_LONG_LONG
__SGI_STL_DECLARE_COPY_TRIVIAL(long long)
__SGI_STL_DECLARE_COPY_TRIVIAL(unsigned long long)
#endif
__SGI_STL_DECLARE_COPY_TRIVIAL(float)
__SGI_STL_DECLARE_COPY_TRIVIAL(double)
__SGI_STL_DECLARE_COPY_TRIVIAL(long double)

#undef __SGI_STL_DECLARE_COPY_TRIVIAL
#endif /* __STL_CLASS_PARTIAL_SPECIALIZATION */
```

以上并没有使用数据类型萃取机制，不知道是不是和编译器类型有关。有些编译器无法识别用户自定义的类型是否具有简单的拷贝构造函数。对于那些有能力识别自定义类型是否有简单拷贝构造函数的编译器而言，实现方式就有所不同了：

```c++
template <class _InputIter, class _OutputIter>
inline _OutputIter copy(_InputIter __first, _InputIter __last,
                        _OutputIter __result) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  typedef typename iterator_traits<_InputIter>::value_type _Tp;  // 迭代器指向的数据类型
  typedef typename __type_traits<_Tp>::has_trivial_assignment_operator
          _Trivial;  // 萃取数据类型是否有简单的拷贝构造函数
  return __copy_dispatch<_InputIter, _OutputIter, _Trivial>
    ::copy(__first, __last, __result);
}
```

```c++
template <class _InputIter, class _OutputIter, class _BoolType>  // false type，没有简单拷贝构造函数
struct __copy_dispatch {
  static _OutputIter copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result) {
    typedef typename iterator_traits<_InputIter>::iterator_category _Category;
    typedef typename iterator_traits<_InputIter>::difference_type _Distance;
    return __copy(__first, __last, __result, _Category(), (_Distance*) 0);
  }
};

template <class _Tp>
struct __copy_dispatch<_Tp*, _Tp*, __true_type>  // true type，有简单拷贝构造函数
{
  static _Tp* copy(const _Tp* __first, const _Tp* __last, _Tp* __result) {
    return __copy_trivial(__first, __last, __result);
  }
};

template <class _Tp>
inline _Tp*
__copy_trivial(const _Tp* __first, const _Tp* __last, _Tp* __result) {
  memmove(__result, __first, sizeof(_Tp) * (__last - __first));  // 直接使用 memmove 拷贝
  return __result + (__last - __first);
}
```

## copy backward

`copy_backward()` 和 `copy()` 几乎相同，唯一的区别是，拷贝是从被拷贝区间的最后一个元素开始，拷贝到目标区间的最后一个位置。因此，拷贝和目标区间可以重叠，但是目标区间的位置需要在拷贝区间之后，否则可能会有问题。

另外，由于是从后向前拷贝，那么迭代器需要支持 `operator--`，迭代器类型需要是 Bidirectional Iterator。

```c++
//--------------------------------------------------
// copy_backward

template <class _BidirectionalIter1, class _BidirectionalIter2,
          class _Distance>
inline _BidirectionalIter2 __copy_backward(_BidirectionalIter1 __first,
                                           _BidirectionalIter1 __last,
                                           _BidirectionalIter2 __result,
                                           bidirectional_iterator_tag,
                                           _Distance*)
{
  while (__first != __last)   // 从 last 开始复制到 first
    *--__result = *--__last;  // 复制的目标位置为 result，result 迭代器不断前移
  return __result;
}

// 针对 Random Access Iterator 的特化版本，提升性能
template <class _RandomAccessIter, class _BidirectionalIter, class _Distance>
inline _BidirectionalIter __copy_backward(_RandomAccessIter __first,
                                          _RandomAccessIter __last,
                                          _BidirectionalIter __result,
                                          random_access_iterator_tag,
                                          _Distance*)
{
  for (_Distance __n = __last - __first; __n > 0; --__n)
    *--__result = *--__last;
  return __result;
}
```
