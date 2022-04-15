# Chapter 6.7.1 - 数据处理算法

Created by : Mr Dk.

2021 / 04 / 14 22:21

Nanjing, Jiangsu, China

---

以下算法只包含单纯的数据移动、线性查找、计数、遍历、遍历施加操作等操作。从操作目标上来说，算法包含两种版本：

- 在指定区间范围上 **就地操作**
- 在指定区间范围上操作后，复制到另一个目标区间中 (通常来说实现上会简单些)

从操作方式上来说，算法也可以分为两个版本：

- 使用默认的方式 (`operator<` 或 `operator==`) 操作数据
- 使用用户自定义的仿函数操作数据

## adjacent_find

找出第一组 **满足条件** 的相邻元素，这里的满足条件指的是：

- (默认) 两个相邻元素相等 (`operator==` 返回 `true`)
- 用户指定一个二元仿函数，操作数为一对相邻元素，函数返回 `true`

一次线性遍历可以实现。

```c++
template <class _ForwardIter>
_ForwardIter adjacent_find(_ForwardIter __first, _ForwardIter __last) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES(typename iterator_traits<_ForwardIter>::value_type,
                 _EqualityComparable);
  if (__first == __last)  // 区间长度为 0，没有相邻元素，直接返回尾迭代器
    return __last;
  _ForwardIter __next = __first;
  while(++__next != __last) {  // first 指向前一个元素，next 指向下一个元素
    if (*__first == *__next)   // 前一个元素和后一个元素相等，返回前一个元素的迭代器位置
      return __first;
    __first = __next;          // first 指向 next，next 指向下一个元素
  }
  return __last;  // 没找到，返回尾迭代器
}

template <class _ForwardIter, class _BinaryPredicate>
_ForwardIter adjacent_find(_ForwardIter __first, _ForwardIter __last,
                           _BinaryPredicate __binary_pred) {  // 用户指定的二元仿函数
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_BINARY_FUNCTION_CHECK(_BinaryPredicate, bool,
          typename iterator_traits<_ForwardIter>::value_type,
          typename iterator_traits<_ForwardIter>::value_type);
  if (__first == __last)
    return __last;
  _ForwardIter __next = __first;
  while(++__next != __last) {
    if (__binary_pred(*__first, *__next))  // 二元仿函数 (参数分别为前一个元素和后一个元素) 返回 true
      return __first;
    __first = __next;
  }
  return __last;
}
```

## find / find_if

找出区间内第一个满足条件的元素，这里的满足条件指的是：

- (默认) 与给定参数相等
- 用户指定的一元仿函数对当前元素返回 `true`

一次线性遍历可以实现。

```c++
template <class _InputIter, class _Tp>
inline _InputIter find(_InputIter __first, _InputIter __last,
                       const _Tp& __val,
                       input_iterator_tag)
{
  while (__first != __last && !(*__first == __val))  // 元素和指定值相等，跳出
    ++__first;
  return __first;
}

template <class _InputIter, class _Predicate>
inline _InputIter find_if(_InputIter __first, _InputIter __last,
                          _Predicate __pred,
                          input_iterator_tag)
{
  while (__first != __last && !__pred(*__first))  // 用户指定的仿函数对当前元素返回 true，跳出
    ++__first;
  return __first;
}
```

## search

在序列一所在的区间中，查找序列二第一次出现的位置。如果序列一中不存在包含序列二的区间，那么返回序列一的尾迭代器。

- 默认使用 `operator==` 来判断序列二的元素是否出现在序列一中
- 用户可以自行指定二元仿函数覆盖默认行为

需要两层循环实现。外层循环遍历序列一，内层循环遍历序列二。

```c++
template <class _ForwardIter1, class _ForwardIter2, class _BinaryPred>
_ForwardIter1 search(_ForwardIter1 __first1, _ForwardIter1 __last1,
                     _ForwardIter2 __first2, _ForwardIter2 __last2,
                     _BinaryPred  __predicate);  // 用户自行提供二元仿函数

template <class _ForwardIter1, class _ForwardIter2>
_ForwardIter1 search(_ForwardIter1 __first1, _ForwardIter1 __last1,
                     _ForwardIter2 __first2, _ForwardIter2 __last2)
{
  __STL_REQUIRES(_ForwardIter1, _ForwardIterator);
  __STL_REQUIRES(_ForwardIter2, _ForwardIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
   typename iterator_traits<_ForwardIter1>::value_type,
   typename iterator_traits<_ForwardIter2>::value_type);

  // Test for empty ranges
  if (__first1 == __last1 || __first2 == __last2)  // 两个序列之一为空，直接结束
    return __first1;

  // Test for a pattern of length 1.
  _ForwardIter2 __tmp(__first2);
  ++__tmp;
  if (__tmp == __last2)
    return find(__first1, __last1, *__first2);  // 带寻找的序列长度为 1，那么等同于在序列中寻找单个元素

  // General case.

  _ForwardIter2 __p1, __p;

  __p1 = __first2; ++__p1;

  _ForwardIter1 __current = __first1;  // current 指向目前正在匹配的序列 1 位置

  while (__first1 != __last1) {  // 序列 1 还未匹配完
    __first1 = find(__first1, __last1, *__first2);  // first1 指向与序列 2 第一个元素匹配的位置
    if (__first1 == __last1)
      return __last1;

    __p = __p1;            // p 指向序列 2 的第一个位置
    __current = __first1;  // current 指向目前正在匹配的序列 1 位置
    if (++__current == __last1)
      return __last1;

    while (*__current == *__p) {  // 序列 1 与序列 2 匹配
      if (++__p == __last2)       // 序列 2 到头，匹配成功
        return __first1;
      if (++__current == __last1) // 序列 1 到头，返回序列 1 结尾的位置
        return __last1;
    }

    ++__first1;  // 下一个 find 区间
  }
  return __first1;
}
```

## search_n

`search_n()` 查找中连续 `n` 个元素的位置：

- 默认使用 `operator==` 来比较序列内元素是否等于给定元素
- 用户可自行提供二元仿函数

代码上写的是两层循环，实际上序列内元素只会被遍历一次。

```c++
template <class _ForwardIter, class _Integer, class _Tp, class _BinaryPred>
_ForwardIter search_n(_ForwardIter __first, _ForwardIter __last,
                      _Integer __count, const _Tp& __val,
                      _BinaryPred __binary_pred);  // 用户自行提供二元仿函数

template <class _ForwardIter, class _Integer, class _Tp>
_ForwardIter search_n(_ForwardIter __first, _ForwardIter __last,
                      _Integer __count, const _Tp& __val) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES(typename iterator_traits<_ForwardIter>::value_type,
                 _EqualityComparable);
  __STL_REQUIRES(_Tp, _EqualityComparable);

  if (__count <= 0)
    return __first;
  else {
    __first = find(__first, __last, __val);  // 找到第一个和 val 相等的位置
    while (__first != __last) {
      _Integer __n = __count - 1;
      _ForwardIter __i = __first;
      ++__i;
      while (__i != __last && __n != 0 && *__i == __val) {
        ++__i;  // 寻找从该位置开始是否存在连续 count 个元素等于 val (operator== 可换为二元仿函数)
        --__n;
      }
      if (__n == 0)
        return __first;  // 满足 count 个元素等于 val
      else
        __first = find(__i, __last, __val);  // 若不满足，则寻找下一个等于 val 的元素出现的位置
    }
    return __last;
  }
}
```

## find_end

寻找序列一所在区间中，序列二最后一次出现的位置。

- (默认) 使用 `operator==` 决定元素是否出现
- 用户可自行提供仿函数

显然，需要两层循环结构来实现。如果迭代器具有 **逆向移动** 的功能，那么相当于在逆向上进行一次 `search()`；否则，迭代器只能从头开始寻找。所以，这里需要根据迭代器的类型做两种不同的实现。这也是 STL 中经常使用的编译器参数推导技巧。

```c++
template <class _ForwardIter1, class _ForwardIter2,
          class _BinaryPredicate>
inline _ForwardIter1
find_end(_ForwardIter1 __first1, _ForwardIter1 __last1,
         _ForwardIter2 __first2, _ForwardIter2 __last2,
         _BinaryPredicate __comp);  // 用户自行提供二元仿函数

template <class _ForwardIter1, class _ForwardIter2>
inline _ForwardIter1
find_end(_ForwardIter1 __first1, _ForwardIter1 __last1,
         _ForwardIter2 __first2, _ForwardIter2 __last2)
{
  __STL_REQUIRES(_ForwardIter1, _ForwardIterator);
  __STL_REQUIRES(_ForwardIter2, _ForwardIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
   typename iterator_traits<_ForwardIter1>::value_type,
   typename iterator_traits<_ForwardIter2>::value_type);
  return __find_end(__first1, __last1, __first2, __last2,
                    __ITERATOR_CATEGORY(__first1),   // 根据迭代器类型进行分派
                    __ITERATOR_CATEGORY(__first2));
}
```

单向 (前向) 迭代器版本：

```c++
// find_end, with and without an explicitly supplied comparison function.
// Search [first2, last2) as a subsequence in [first1, last1), and return
// the *last* possible match.  Note that find_end for bidirectional iterators
// is much faster than for forward iterators.

// find_end for forward iterators.
template <class _ForwardIter1, class _ForwardIter2>
_ForwardIter1 __find_end(_ForwardIter1 __first1, _ForwardIter1 __last1,
                         _ForwardIter2 __first2, _ForwardIter2 __last2,
                         forward_iterator_tag, forward_iterator_tag)  、、
{
  if (__first2 == __last2)
    return __last1;
  else {
    _ForwardIter1 __result = __last1;
    while (1) {
      _ForwardIter1 __new_result  // 找到序列 1 中的下一个序列 2 子区间
        = search(__first1, __last1, __first2, __last2);
      if (__new_result == __last1)  // 序列 1 到头
        return __result;            // 返回目前已经找到的最后一个有效子区间
      else {                      // 新找到的子区间有效
        __result = __new_result;  // 将最后匹配的子区间更新为这个子区间
        __first1 = __new_result;  // 寻找下一个可能的子区间
        ++__first1;
      }
    }
  }
}
```

双向迭代器版本 (快很多)：

```c++
// find_end for bidirectional iterators.  Requires partial specialization.

template <class _BidirectionalIter1, class _BidirectionalIter2>
_BidirectionalIter1
__find_end(_BidirectionalIter1 __first1, _BidirectionalIter1 __last1,
           _BidirectionalIter2 __first2, _BidirectionalIter2 __last2,
           bidirectional_iterator_tag, bidirectional_iterator_tag)
{
  __STL_REQUIRES(_BidirectionalIter1, _BidirectionalIterator);
  __STL_REQUIRES(_BidirectionalIter2, _BidirectionalIterator);
  typedef reverse_iterator<_BidirectionalIter1> _RevIter1;
  typedef reverse_iterator<_BidirectionalIter2> _RevIter2;

  _RevIter1 __rlast1(__first1);
  _RevIter2 __rlast2(__first2);
  _RevIter1 __rresult = search(_RevIter1(__last1), __rlast1,   // 转换为反向迭代器，并调用 search
                               _RevIter2(__last2), __rlast2);

  if (__rresult == __rlast1)  // 没找到，返回序列 1 的尾迭代器
    return __last1;
  else {                                              // 区间找到
    _BidirectionalIter1 __result = __rresult.base();  // 实际上对应匹配上区间的尾部
    advance(__result, -distance(__first2, __last2));  // 区间尾部减去区间长度，就是区间开始的位置
    return __result;
  }
}
```

## find_first_of

寻找序列 1 中，第一次出现序列 2 中任意元素的位置。显然这也会是一个二层循环。

- (默认) 使用 `operator==` 决定元素是否出现
- 用户自行提供二元仿函数

```c++
// find_first_of, with and without an explicitly supplied comparison function.

template <class _InputIter, class _ForwardIter, class _BinaryPredicate>
_InputIter find_first_of(_InputIter __first1, _InputIter __last1,
                         _ForwardIter __first2, _ForwardIter __last2,
                         _BinaryPredicate __comp);  // 用户提供的二元仿函数

template <class _InputIter, class _ForwardIter>
_InputIter find_first_of(_InputIter __first1, _InputIter __last1,
                         _ForwardIter __first2, _ForwardIter __last2)
{
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
     typename iterator_traits<_InputIter>::value_type,
     typename iterator_traits<_ForwardIter>::value_type);

  for ( ; __first1 != __last1; ++__first1)  // 外层循环遍历序列 1
    for (_ForwardIter __iter = __first2; __iter != __last2; ++__iter)  // 内存循环遍历序列 2
      if (*__first1 == *__iter)  // 这里可被替换为用户自定义的仿函数
        return __first1;  // 找到，立刻返回
  return __last1;  // 没找到
}
```

## count / count_if

返回区间内与指定值相等的元素个数，泛化后用户可以提供一个一元仿函数，返回仿函数应用在元素上返回 `true` 的元素个数。遍历一次区间即可。

```c++
// count and count_if.  There are two version of each, one whose return type
// type is void and one (present only if we have partial specialization)
// whose return type is iterator_traits<_InputIter>::difference_type.  The
// C++ standard only has the latter version, but the former, which was present
// in the HP STL, is retained for backward compatibility.

template <class _InputIter, class _Tp, class _Size>
void count(_InputIter __first, _InputIter __last, const _Tp& __value,
           _Size& __n) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(typename iterator_traits<_InputIter>::value_type,
                 _EqualityComparable);
  __STL_REQUIRES(_Tp, _EqualityComparable);
  for ( ; __first != __last; ++__first)
    if (*__first == __value)  // 可被替换为一元仿函数
      ++__n;
}

template <class _InputIter, class _Predicate, class _Size>
void count_if(_InputIter __first, _InputIter __last, _Predicate __pred,  // 用户提供一元仿函数
              _Size& __n);
```

## for_each

将仿函数施加在区间内的每一个元素身上，仿函数的返回值被忽略。由于迭代器参数只读 (Input Iterator)，仿函数不能修改区间内元素。

```c++
// for_each.  Apply a function to every element of a range.
template <class _InputIter, class _Function>
_Function for_each(_InputIter __first, _InputIter __last, _Function __f) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  for ( ; __first != __last; ++__first)
    __f(*__first);  // 仿函数返回值被忽略
  return __f;
}
```

## transform

将 (一元 / 二元) 仿函数施加在区间内的 (每一个 / 每一对) 元素身上，并将返回值输出到一个区间中。输出区间可以是输入区间，那么仿函数的参数将会被仿函数的返回值替换。

```c++
// transform

template <class _InputIter, class _OutputIter, class _UnaryOperation>
_OutputIter transform(_InputIter __first, _InputIter __last,
                      _OutputIter __result, _UnaryOperation __opr) {  // 一元仿函数
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);

  for ( ; __first != __last; ++__first, ++__result)
    *__result = __opr(*__first);
  return __result;
}

template <class _InputIter1, class _InputIter2, class _OutputIter,
          class _BinaryOperation>
_OutputIter transform(_InputIter1 __first1, _InputIter1 __last1,
                      _InputIter2 __first2, _OutputIter __result,
                      _BinaryOperation __binary_op) {  // 二元仿函数
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  for ( ; __first1 != __last1; ++__first1, ++__first2, ++__result)
    *__result = __binary_op(*__first1, *__first2);
  return __result;
}
```

## generate / generate_n

将用户提供的仿函数的运算结果赋值到区间内所有 (前 `n` 个) 元素上。

```c++
// generate and generate_n

template <class _ForwardIter, class _Generator>
void generate(_ForwardIter __first, _ForwardIter __last, _Generator __gen) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_GENERATOR_CHECK(_Generator,
          typename iterator_traits<_ForwardIter>::value_type);
  for ( ; __first != __last; ++__first)
    *__first = __gen();  // 赋值
}

template <class _OutputIter, class _Size, class _Generator>
_OutputIter generate_n(_OutputIter __first, _Size __n, _Generator __gen) {
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  for ( ; __n > 0; --__n, ++__first)
    *__first = __gen();  // 赋值 (同时计数 n)
  return __first;
}
```

## max_element / min_element

遍历序列一次，返回序列中最大或最小值的位置。

```c++
// min_element and max_element, with and without an explicitly supplied
// comparison function.

template <class _ForwardIter>
_ForwardIter max_element(_ForwardIter __first, _ForwardIter __last) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES(typename iterator_traits<_ForwardIter>::value_type,
                 _LessThanComparable);
  if (__first == __last) return __first;
  _ForwardIter __result = __first;
  while (++__first != __last)
    if (*__result < *__first)  // 大于现有最大值
      __result = __first;
  return __result;
}

template <class _ForwardIter, class _Compare>
_ForwardIter max_element(_ForwardIter __first, _ForwardIter __last,
			 _Compare __comp);

template <class _ForwardIter>
_ForwardIter min_element(_ForwardIter __first, _ForwardIter __last) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES(typename iterator_traits<_ForwardIter>::value_type,
                 _LessThanComparable);
  if (__first == __last) return __first;
  _ForwardIter __result = __first;
  while (++__first != __last)
    if (*__first < *__result)  // 小于现有最小值
      __result = __first;
  return __result;
}

template <class _ForwardIter, class _Compare>
_ForwardIter min_element(_ForwardIter __first, _ForwardIter __last,
			 _Compare __comp);
```

## remove / remove_copy / remove_if / remove_copy_if

`remove_copy()` 移除区间内所有与给定值相等的元素，不从区间中真正删除元素，而是将结果复制到一个特定空间中。操作空间和目标空间可以是同一个。`remove_copy_if()` 使用用户自定义的一元仿函数替换 `operator==`，删除使仿函数返回 `true` 的元素。

```c++
template <class _InputIter, class _OutputIter, class _Tp>
_OutputIter remove_copy(_InputIter __first, _InputIter __last,
                        _OutputIter __result, const _Tp& __value) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
       typename iterator_traits<_InputIter>::value_type, _Tp);
  for ( ; __first != __last; ++__first)
    if (!(*__first == __value)) {  // 如果当前元素 != 给定值
      *__result = *__first;        // 将当前元素复制到目标位置
      ++__result;                  // 目标位置指向下一个位置
    }
  return __result;
}

template <class _InputIter, class _OutputIter, class _Predicate>
_OutputIter remove_copy_if(_InputIter __first, _InputIter __last,
                           _OutputIter __result, _Predicate __pred) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_UNARY_FUNCTION_CHECK(_Predicate, bool,
             typename iterator_traits<_InputIter>::value_type);
  for ( ; __first != __last; ++__first)
    if (!__pred(*__first)) {
      *__result = *__first;
      ++__result;
    }
  return __result;
}
```

`remove()` 借用了 `remove_copy()` 的实现，将结果直接覆盖在当前容器中；`remove_if()` 借用了 `remove_copy_if()` 的实现，将结果覆盖在当前容器中。

```c++
template <class _ForwardIter, class _Tp>
_ForwardIter remove(_ForwardIter __first, _ForwardIter __last,
                    const _Tp& __value) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
       typename iterator_traits<_ForwardIter>::value_type, _Tp);
  __STL_CONVERTIBLE(_Tp, typename iterator_traits<_ForwardIter>::value_type);
  __first = find(__first, __last, __value);  // 找到区间内第一个删除位置
  _ForwardIter __i = __first;
  return __first == __last ? __first
                           : remove_copy(++__i, __last, __first, __value);  // 从删除位置的下一个位置开始 remove_copy()
}

template <class _ForwardIter, class _Predicate>
_ForwardIter remove_if(_ForwardIter __first, _ForwardIter __last,
                       _Predicate __pred) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_UNARY_FUNCTION_CHECK(_Predicate, bool,
               typename iterator_traits<_ForwardIter>::value_type);
  __first = find_if(__first, __last, __pred);  // 调用 find_if() 找到第一个删除位置
  _ForwardIter __i = __first;
  return __first == __last ? __first
                           : remove_copy_if(++__i, __last, __first, __pred);
}
```

## replace / replace_copy / replace_if / replace_copy_if

`replace()` 将区间内的特定值用新值替代，使用 `operator==` 来判断元素是否等于某个特定值；`replace_if()` 使用用户提供的一元仿函数作用在元素上是否返回 `true` 来决定是否被替换。

```c++
// replace, replace_if, replace_copy, replace_copy_if

template <class _ForwardIter, class _Tp>
void replace(_ForwardIter __first, _ForwardIter __last,
             const _Tp& __old_value, const _Tp& __new_value) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
         typename iterator_traits<_ForwardIter>::value_type, _Tp);
  __STL_CONVERTIBLE(_Tp, typename iterator_traits<_ForwardIter>::value_type);
  for ( ; __first != __last; ++__first)
    if (*__first == __old_value)  // 元素 == 指定旧值
      *__first = __new_value;     // 元素被替换为新值
}

template <class _ForwardIter, class _Predicate, class _Tp>
void replace_if(_ForwardIter __first, _ForwardIter __last,
                _Predicate __pred, const _Tp& __new_value) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_CONVERTIBLE(_Tp, typename iterator_traits<_ForwardIter>::value_type);
  __STL_UNARY_FUNCTION_CHECK(_Predicate, bool,
             typename iterator_traits<_ForwardIter>::value_type);
  for ( ; __first != __last; ++__first)
    if (__pred(*__first))      // 仿函数作用域元素上返回 true
      *__first = __new_value;  // 元素被替换为新值
}
```

以下两个函数行为类似，一边将序列中的元素复制到一个目标空间中，一边将特定值替换为新值。

```c++
template <class _InputIter, class _OutputIter, class _Tp>
_OutputIter replace_copy(_InputIter __first, _InputIter __last,
                         _OutputIter __result,
                         const _Tp& __old_value, const _Tp& __new_value) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES_BINARY_OP(_OP_EQUAL, bool,
         typename iterator_traits<_InputIter>::value_type, _Tp);
  for ( ; __first != __last; ++__first, ++__result)
    *__result = *__first == __old_value ? __new_value : *__first;  // 复制原元素，或替换新值
  return __result;
}

template <class _InputIter, class _OutputIter, class _Predicate, class _Tp>
_OutputIter replace_copy_if(_InputIter __first, _InputIter __last,
                            _OutputIter __result,
                            _Predicate __pred, const _Tp& __new_value) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_UNARY_FUNCTION_CHECK(_Predicate, bool,
                typename iterator_traits<_InputIter>::value_type);
  for ( ; __first != __last; ++__first, ++__result)
    *__result = __pred(*__first) ? __new_value : *__first;  // 复制原元素，或替换新值
  return __result;
}
```

## reverse / reverse_copy

`reverse()` 将区间内的元素就地颠倒顺序；`reverse_copy()` 将区间内的元素在一个新的区间内颠倒顺序。`reverse_copy()` 的实现相对来说简单一些，通过双向迭代器依次将区间内元素从后到前复制即可。

```c++
template <class _BidirectionalIter, class _OutputIter>
_OutputIter reverse_copy(_BidirectionalIter __first,
                         _BidirectionalIter __last,
                         _OutputIter __result) {
  __STL_REQUIRES(_BidirectionalIter, _BidirectionalIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  while (__first != __last) {
    --__last;             // 迭代器从后到前移动
    *__result = *__last;  // 复制
    ++__result;
  }
  return __result;
}
```

而对于就地颠倒来说，双向迭代器和随机存取迭代器的实现方式不同：

```c++
template <class _BidirectionalIter>
inline void reverse(_BidirectionalIter __first, _BidirectionalIter __last) {
  __STL_REQUIRES(_BidirectionalIter, _Mutable_BidirectionalIterator);
  __reverse(__first, __last, __ITERATOR_CATEGORY(__first));  // 根据迭代器类型分派
}

template <class _BidirectionalIter>
void __reverse(_BidirectionalIter __first, _BidirectionalIter __last,
               bidirectional_iterator_tag) {  // 双向迭代器版本
  while (true)
    if (__first == __last || __first == --__last)  // 循环终止条件为首尾迭代器相同或交错
      return;
    else
      iter_swap(__first++, __last);
}

template <class _RandomAccessIter>
void __reverse(_RandomAccessIter __first, _RandomAccessIter __last,
               random_access_iterator_tag) {  // 随机存取迭代器版本
  while (__first < __last)  // 循环终止条件为迭代器相遇 (随机存取迭代器可比较)
    iter_swap(__first++, --__last);
}
```

## rotate / rotate_copy

将一段区间内 `middle` 之前与 `middle` 之后的元素进行颠倒，每个子区间内的元素顺序不变。

`rotate_copy()` 将颠倒后的结果保存到一个目标区间中，因此实现较为简单；`rotate()` 就地交换两个子区间，实现上根据迭代器类型的不同而不同。

```c++
template <class _ForwardIter, class _OutputIter>
_OutputIter rotate_copy(_ForwardIter __first, _ForwardIter __middle,
                        _ForwardIter __last, _OutputIter __result) {
  __STL_REQUIRES(_ForwardIter, _ForwardIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  return copy(__first, __middle, copy(__middle, __last, __result));
  // 先拷贝 middle 到 last 的元素到目标区间，然后拷贝 first 到 middle 的元素到目标区间
}
```

```c++
template <class _ForwardIter>
inline _ForwardIter rotate(_ForwardIter __first, _ForwardIter __middle,
                           _ForwardIter __last) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  return __rotate(__first, __middle, __last,
                  __DISTANCE_TYPE(__first),
                  __ITERATOR_CATEGORY(__first));  // 根据迭代器类型分派
}

template <class _BidirectionalIter, class _Distance>
_BidirectionalIter __rotate(_BidirectionalIter __first,
                            _BidirectionalIter __middle,
                            _BidirectionalIter __last,
                            _Distance*,
                            bidirectional_iterator_tag) {  // 双向迭代器版本
  __STL_REQUIRES(_BidirectionalIter, _Mutable_BidirectionalIterator);
  if (__first == __middle)
    return __last;
  if (__last  == __middle)
    return __first;

  __reverse(__first,  __middle, bidirectional_iterator_tag());  // 颠倒 first 到 middle 之间的区间
  __reverse(__middle, __last,   bidirectional_iterator_tag());  // 颠倒 middle 到 last 之间的区间

  // 以下，颠倒 first 到 last 之间的区间

  while (__first != __middle && __middle != __last)
    swap (*__first++, *--__last);

  if (__first == __middle) {
    __reverse(__middle, __last,   bidirectional_iterator_tag());
    return __last;
  }
  else {
    __reverse(__first,  __middle, bidirectional_iterator_tag());
    return __first;
  }
}
```

> 单向迭代器和随机访问迭代器版本的代码无法消化... 😔

## swap_ranges

功能与 `rotate()` 类似，但是只能交换元素个数相同的区间。通过循环，交换两对迭代器指向的对应元素。

```c++
// swap_ranges

template <class _ForwardIter1, class _ForwardIter2>
_ForwardIter2 swap_ranges(_ForwardIter1 __first1, _ForwardIter1 __last1,
                          _ForwardIter2 __first2) {
  __STL_REQUIRES(_ForwardIter1, _Mutable_ForwardIterator);
  __STL_REQUIRES(_ForwardIter2, _Mutable_ForwardIterator);
  __STL_CONVERTIBLE(typename iterator_traits<_ForwardIter1>::value_type,
                    typename iterator_traits<_ForwardIter2>::value_type);
  __STL_CONVERTIBLE(typename iterator_traits<_ForwardIter2>::value_type,
                    typename iterator_traits<_ForwardIter1>::value_type);
  for ( ; __first1 != __last1; ++__first1, ++__first2)
    iter_swap(__first1, __first2);  // 交换两对迭代器指向的元素
  return __first2;
}
```

## unique / unique_copy

移除 **相邻的重复元素**。如果想要移除所有 (包括不相邻) 的重复元素，需要先对序列排序，使所有重复元素相邻。

`unique()` 返回迭代器指向新区间的尾部，迭代器之后的元素是一些保留下来的残余元素。元素是否重复默认由 `operator==` 定义，但用户可以自行提供一个二元仿函数。

`unique_copy()` 将元素复制到另一个区间上，如果遇到相邻的重复元素，则只会复制一个。

```c++
template <class _InputIter, class _OutputIter>
inline _OutputIter unique_copy(_InputIter __first, _InputIter __last,
                               _OutputIter __result) {
  __STL_REQUIRES(_InputIter, _InputIterator);
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  __STL_REQUIRES(typename iterator_traits<_InputIter>::value_type,
                 _EqualityComparable);
  if (__first == __last) return __result;
  return __unique_copy(__first, __last, __result,
                       __ITERATOR_CATEGORY(__result));  // 根据迭代器类型分派
}

///////////////////////////////////////////////////////////////////////////

template <class _InputIter, class _OutputIter, class _Tp>
_OutputIter __unique_copy(_InputIter __first, _InputIter __last,
                          _OutputIter __result, _Tp*) {
  _Tp __value = *__first;
  *__result = __value;
  while (++__first != __last)
    if (!(__value == *__first)) {  // 输出迭代器不可读
      __value = *__first;
      *++__result = __value;       // 因此需要用一个变量保存输出迭代器对应的值
    }
  return ++__result;
}

template <class _InputIter, class _OutputIter>
inline _OutputIter __unique_copy(_InputIter __first, _InputIter __last,
                                 _OutputIter __result,
                                 output_iterator_tag) {  // 输出迭代器版本
  return __unique_copy(__first, __last, __result, __VALUE_TYPE(__first));
}

///////////////////////////////////////////////////////////////////////////

template <class _InputIter, class _ForwardIter>
_ForwardIter __unique_copy(_InputIter __first, _InputIter __last,
                           _ForwardIter __result, forward_iterator_tag) {  // 单向迭代器版本
  *__result = *__first;  // 复制第一个元素
  while (++__first != __last)
    if (!(*__result == *__first))  // 目标区间最后一个元素 != 当前元素
      *++__result = *__first;      // 向目标区间复制当前元素
  return ++__result;
}
```

`unique()` 直接借用了 `unique_copy()` 的实现：

```c++
template <class _ForwardIter>
_ForwardIter unique(_ForwardIter __first, _ForwardIter __last) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_REQUIRES(typename iterator_traits<_ForwardIter>::value_type,
                 _EqualityComparable);
  __first = adjacent_find(__first, __last);
  return unique_copy(__first, __last, __first);
}

template <class _ForwardIter, class _BinaryPredicate>
_ForwardIter unique(_ForwardIter __first, _ForwardIter __last,
                    _BinaryPredicate __binary_pred);
```

## includes

判断序列一中是否包含序列二 (序列二中的每个元素都出现在序列一中)。两个序列 **必须有序**。根据两个序列递增 / 递减，算法使用 `less / `greater` 仿函数进行比较。用户也可以自行提供一个用于比较的二元仿函数。

```c++
template <class _InputIter1, class _InputIter2>
bool includes(_InputIter1 __first1, _InputIter1 __last1,
              _InputIter2 __first2, _InputIter2 __last2) {
  __STL_REQUIRES(_InputIter1, _InputIterator);
  __STL_REQUIRES(_InputIter2, _InputIterator);
  __STL_REQUIRES_SAME_TYPE(
       typename iterator_traits<_InputIter1>::value_type,
       typename iterator_traits<_InputIter2>::value_type);
  __STL_REQUIRES(typename iterator_traits<_InputIter1>::value_type,
                 _LessThanComparable);
  while (__first1 != __last1 && __first2 != __last2)  // 两个序列未到结尾
    if (*__first2 < *__first1)      // 序列二中的元素 < 序列一中的元素
      return false;                 // 包含不成立
    else if(*__first1 < *__first2)  // 序列一中的元素 < 序列二中的元素
      ++__first1;                   // 序列一的下一个元素
    else                            // 序列一中的元素 == 序列二中的元素
      ++__first1, ++__first2;       // 比较两个序列中的下一个元素

  return __first2 == __last2;  // 序列二是否已到尾部
}

template <class _InputIter1, class _InputIter2, class _Compare>
bool includes(_InputIter1 __first1, _InputIter1 __last1,
              _InputIter2 __first2, _InputIter2 __last2, _Compare __comp);
```

## merge

将两个排序后的区间有序合并到另一段区间中。用户可指定比较仿函数替代默认的 `operator<`。

```c++
// merge, with and without an explicitly supplied comparison function.

template <class _InputIter1, class _InputIter2, class _OutputIter>
_OutputIter merge(_InputIter1 __first1, _InputIter1 __last1,
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
  while (__first1 != __last1 && __first2 != __last2) {  // 两个序列没到结尾
    if (*__first2 < *__first1) {  // 序列二中的元素 < 序列一中的元素
      *__result = *__first2;      // 复制序列二中的元素到目标区间
      ++__first2;
    }
    else {                        // 序列一中的元素 <= 序列二中的元素
      *__result = *__first1;      // 复制序列一中的元素到目标区间
      ++__first1;
    }
    ++__result;
  }
  return copy(__first2, __last2, copy(__first1, __last1, __result));  // 复制序列一或序列二中的剩余元素
}

template <class _InputIter1, class _InputIter2, class _OutputIter,
          class _Compare>
_OutputIter merge(_InputIter1 __first1, _InputIter1 __last1,
                  _InputIter2 __first2, _InputIter2 __last2,
                  _OutputIter __result, _Compare __comp);
```

## partition

将区间内的元素重新排列，排列依据是提供的一元仿函数作用在元素上是否返回 `true`，所有返回 `true` 的元素将排列在返回 `false` 的元素之前。该算法 **不稳定**。想象如果这个一元仿函数是 `less`，那么效果就是所有小于某个值的元素将会排列在所有不小于该值的元素之前 (快速排序的必要算法)。

```c++
template <class _ForwardIter, class _Predicate>
inline _ForwardIter partition(_ForwardIter __first,
   			      _ForwardIter __last,
			      _Predicate   __pred) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_UNARY_FUNCTION_CHECK(_Predicate, bool,
        typename iterator_traits<_ForwardIter>::value_type);
  return __partition(__first, __last, __pred, __ITERATOR_CATEGORY(__first));  // 根据迭代器类型分派
}

template <class _ForwardIter, class _Predicate>
_ForwardIter __partition(_ForwardIter __first,
		         _ForwardIter __last,
			 _Predicate   __pred,
			 forward_iterator_tag) {  // 单向迭代器版本
  if (__first == __last) return __first;

  while (__pred(*__first))
    if (++__first == __last) return __first;

  _ForwardIter __next = __first;  // 第一个不满足 true 的位置

  while (++__next != __last)      // next 不断向后寻找可以满足 true 的元素，并交换到前面的位置
    if (__pred(*__next)) {        // 之后第一个满足 true 的位置
      swap(*__first, *__next);    // 交换元素
      ++__first;                  // first 指向下一个 (可以放置满足 true 的元素) 位置
    }

  return __first;
}

template <class _BidirectionalIter, class _Predicate>
_BidirectionalIter __partition(_BidirectionalIter __first,
                               _BidirectionalIter __last,
			       _Predicate __pred,
			       bidirectional_iterator_tag) {  // 双向迭代器版本
  while (true) {
    while (true)  // 从前开始找
      if (__first == __last)
        return __first;
      else if (__pred(*__first))
        ++__first;
      else
        break;    // 元素不满足 true，跳出
    --__last;
    while (true)  // 从后开始找
      if (__first == __last)
        return __first;
      else if (!__pred(*__last))
        --__last;
      else        // 元素满足 true，跳出
        break;
    iter_swap(__first, __last);  // 交换前后两个不满足条件的元素
    ++__first;
  }
}
```

> 尼玛的，累死我了... 😆
