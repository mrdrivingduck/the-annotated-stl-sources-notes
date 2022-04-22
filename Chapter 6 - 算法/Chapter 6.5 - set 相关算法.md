# Chapter 6.5 - set 相关算法

Created by : Mr Dk.

2021 / 04 / 14 16:14

Nanjing, Jiangsu, China

---

在 STL 中，STL 的定义要求 set 的元素不得重复，并且经过排序。基于这个定义，hash_set 和 hash_multiset 由于不排序，因此不可以应用下面的算法。

set 的这几个相关算法默认都使用了 `operator<`，配合两个序列的元素作为操作数 (两种不同位置)，可以得到两个序列对应元素的小于 / 大于 / 等于三种相对关系，从而实现相应操作。`operator<` 全都可以换成用户自定义的仿函数。

## 6.5.1 set_union

`set_union()` 构造两个 set 的并集，也就是构造一个集合，集合内包含两个 set 内的每一个元素。如果两个 set 都出现了同一个元素，那么每一对相同元素将会对应目标集合中的一个元素 (看代码实现可知)。算法使用 `operator<` 来确定两个元素是否相等 (`a < b` / `b < a`)，也可以接收用户提供的二元仿函数。

```cpp
template <class _InputIter1, class _InputIter2, class _OutputIter>
_OutputIter set_union(_InputIter1 __first1, _InputIter1 __last1,
                      _InputIter2 __first2, _InputIter2 __last2,
                      _OutputIter __result) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _LessThanComparable);
  while (__first1 != __last1 && __first2 != __last2) {  // 未到两个序列的尾部
    if (*__first1 < *__first2) {  // 第一个集合的元素较小
      *__result = *__first1;      // 复制第一个集合的元素到目标集合
      ++__first1;                 // 第一个集合的下一个元素
    }
    else if (*__first2 < *__first1) {  // 第二个集合的元素较小
      *__result = *__first2;           // 复制第二个集合的元素到目标集合
      ++__first2;                      // 第二个集合的下一个元素
    }
    else {                    // 两个集合的元素相等
      *__result = *__first1;  // 复制一个元素到目标集合 (因此一对相等元素对应目标集合的一个元素)
      ++__first1;             // 两个集合的下一个元素
      ++__first2;
    }
    ++__result;
  }
  return copy(__first2, __last2, copy(__first1, __last1, __result));  // 拷贝未到结尾的序列剩余元素
}

template <class _InputIter1, class _InputIter2, class _OutputIter,
          class _Compare>
_OutputIter set_union(_InputIter1 __first1, _InputIter1 __last1,
                      _InputIter2 __first2, _InputIter2 __last2,
                      _OutputIter __result, _Compare __comp) {  // 接收用户提供的二元仿函数
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_BINARY_FUNCTION_CHECK(_Compare, bool,
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  while (__first1 != __last1 && __first2 != __last2) {
    if (__comp(*__first1, *__first2)) {  // operator< 替换为二元仿函数
      *__result = *__first1;
      ++__first1;
    }
    else if (__comp(*__first2, *__first1)) {  // operator< 替换为二元仿函数
      *__result = *__first2;
      ++__first2;
    }
    else {
      *__result = *__first1;
      ++__first1;
      ++__first2;
    }
    ++__result;
  }
  return copy(__first2, __last2, copy(__first1, __last1, __result));
}
```

## 6.5.2 set_intersection

`set_intersection()` 构造两个 set 的交集，即集合内为同时出现在两个 set 内的每一对元素。同样提供两个版本：`operator<` 版本和用户自定义的二元仿函数版本。

```cpp
template <class _InputIter1, class _InputIter2, class _OutputIter>
_OutputIter set_intersection(_InputIter1 __first1, _InputIter1 __last1,
                             _InputIter2 __first2, _InputIter2 __last2,
                             _OutputIter __result) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _LessThanComparable);
  while (__first1 != __last1 && __first2 != __last2)  // 两个集合没到尾部
    if (*__first1 < *__first2)  // 第一个集合的元素比第二个集合的元素小
      ++__first1;               // 第一个集合的下一个元素 (前一个元素不可能在交集中)
    else if (*__first2 < *__first1)  // 第二个集合的元素比第一个集合的元素小
      ++__first2;                    // 第二个集合的下一个元素 (前一个元素不可能在交集中)
    else {                    // 第一个集合的元素与第二个集合的元素相等
      *__result = *__first1;  // 复制该元素到目标集合中
      ++__first1;             // 第一个集合的下一个元素
      ++__first2;             // 第二个集合的下一个元素
      ++__result;             // 目标集合的下一个位置
    }
  return __result;  // 返回目标集合尾迭代器
}

template <class _InputIter1, class _InputIter2, class _OutputIter,
          class _Compare>
_OutputIter set_intersection(_InputIter1 __first1, _InputIter1 __last1,
                             _InputIter2 __first2, _InputIter2 __last2,
                             _OutputIter __result, _Compare __comp) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_BINARY_FUNCTION_CHECK(_Compare, bool,
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);

  while (__first1 != __last1 && __first2 != __last2)
    if (__comp(*__first1, *__first2))  // operator< 替换为二元仿函数
      ++__first1;
    else if (__comp(*__first2, *__first1))  // opeator< 替换为二元仿函数
      ++__first2;
    else {
      *__result = *__first1;
      ++__first1;
      ++__first2;
      ++__result;
    }
  return __result;
}
```

## 6.5.3 set_difference

构造两个集合之间的差集，即出现在第一个集合但没有出现在第二个集合中的所有元素。同样支持 `operator<` 和用户自定义仿函数的实现版本。

```cpp
template <class _InputIter1, class _InputIter2, class _OutputIter>
_OutputIter set_difference(_InputIter1 __first1, _InputIter1 __last1,
                           _InputIter2 __first2, _InputIter2 __last2,
                           _OutputIter __result) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _LessThanComparable);
  while (__first1 != __last1 && __first2 != __last2)  // 两个序列没到结尾
    if (*__first1 < *__first2) {  // 第一个集合中的元素 < 第二个集合中的元素
      *__result = *__first1;      // 这个元素肯定不会出现在第二个集合中，加入差集
      ++__first1;                 // 第一个集合的下一个元素
      ++__result;                 // 目标集合的下一个位置
    }
    else if (*__first2 < *__first1)  // 第二个集合中的元素 < 第一个集合中的元素
      ++__first2;                    // 第二个集合中的下一个元素 (忽略当前元素)
    else {         // 第一个集合中的元素 == 第二个集合中的元素，肯定不在差集中
      ++__first1;  // 第一个集合中的下一个元素
      ++__first2;  // 第二个集合中的下一个元素
    }
  return copy(__first1, __last1, __result);  // 第一个集合中的剩余元素全部复制到差集中
}

template <class _InputIter1, class _InputIter2, class _OutputIter,
          class _Compare>
_OutputIter set_difference(_InputIter1 __first1, _InputIter1 __last1,
                           _InputIter2 __first2, _InputIter2 __last2,
                           _OutputIter __result, _Compare __comp) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_BINARY_FUNCTION_CHECK(_Compare, bool,
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);

  while (__first1 != __last1 && __first2 != __last2)
    if (__comp(*__first1, *__first2)) {  // 将 operator< 替换为二元仿函数
      *__result = *__first1;
      ++__first1;
      ++__result;
    }
    else if (__comp(*__first2, *__first1))  // 将 operator< 替换为二元仿函数
      ++__first2;
    else {
      ++__first1;
      ++__first2;
    }
  return copy(__first1, __last1, __result);
}
```

## 6.5.4 set_symmetric_difference

`set_symmetric_difference()` 可以得到两个集合的对称差集，即 `(S1 - S2) ∪ (S2 - S1)`，_出现在第一个集合但不出现在第二个集合_ 以及 _出现在第二个集合但不出现在第一个集合_ 的每一个元素。参考之前非对称差集的实现 (`set_difference()`) 就可以知道如何实现这个函数。同样，支持 `operator<` 和二元仿函数两个版本。

```cpp
template <class _InputIter1, class _InputIter2, class _OutputIter>
_OutputIter
set_symmetric_difference(_InputIter1 __first1, _InputIter1 __last1,
                         _InputIter2 __first2, _InputIter2 __last2,
                         _OutputIter __result) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _LessThanComparable);
  while (__first1 != __last1 && __first2 != __last2)  // 两个序列都没有到尾部
    if (*__first1 < *__first2) {  // 第一个集合的元素 < 第二个集合的元素
      *__result = *__first1;      // 第一个集合的元素加入对称差集
      ++__first1;                 // 第一个集合的下一个元素
      ++__result;
    }
    else if (*__first2 < *__first1) {  // 第二个集合的元素 < 第一个集合的元素
      *__result = *__first2;           // 第二个集合的元素加入对称差集
      ++__first2;                      // 第二个集合的下一个元素
      ++__result;
    }
    else {         // 第一个集合的元素 == 第二个集合的元素，这对元素肯定不在对称差集中
      ++__first1;  // 第一个集合的下一个元素
      ++__first2;  // 第二个集合的下一个元素
    }
  return copy(__first2, __last2, copy(__first1, __last1, __result));  // 将还未到结尾的集合剩余元素加入到对称差集中
}

template <class _InputIter1, class _InputIter2, class _OutputIter,
          class _Compare>
_OutputIter
set_symmetric_difference(_InputIter1 __first1, _InputIter1 __last1,
                         _InputIter2 __first2, _InputIter2 __last2,
                         _OutputIter __result,
                         _Compare __comp) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_BINARY_FUNCTION_CHECK(_Compare, bool,
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  while (__first1 != __last1 && __first2 != __last2)
    if (__comp(*__first1, *__first2)) {  // operator< 替换为二元仿函数
      *__result = *__first1;
      ++__first1;
      ++__result;
    }
    else if (__comp(*__first2, *__first1)) {  // operator< 替换为二元仿函数
      *__result = *__first2;
      ++__first2;
      ++__result;
    }
    else {
      ++__first1;
      ++__first2;
    }
  return copy(__first2, __last2, copy(__first1, __last1, __result));
}
```
