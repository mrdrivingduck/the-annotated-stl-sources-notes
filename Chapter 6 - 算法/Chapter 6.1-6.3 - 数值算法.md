# Chapter 6.1-6.3 - 数值算法

Created by : Mr Dk.

2021 / 04 / 10 14:58

Nanjing, Jiangsu, China

---

## 算法的泛化

算法的分类：

- 质变算法：会改变操作对象的内容，如排序、删除、替换等
  - 就地进行 (in-place)：就地改变操作对象
  - 拷贝进行：将操作内容复制为一份副本，对副本进行修改并返回副本
- 非质变算法：不改变操作对象的内容，如查找、计数、遍历等

所有泛型算法的前两个参数都是一对迭代器，指示了算法的操作区间 (左闭右开)。对 `[first, last)` 区间进行操作的必要条件是，必须能够由 `operator++` 使迭代器能够从 `first` 前进到 `last`。根据迭代器的前进特性，STL 的每一个算法声明都要表现出 **所需最低程度的迭代器类型**。

许多 STL 算法支持多个版本，其中原始版本使用缺省运算行为，另一个版本支持用户传入 **仿函数**。

## 6.3.2 accumulate

用于将一个区间的值累加到某个初始值身上，并返回累加值。除了加法，可以泛化为其它二元操作：

```cpp
template <class _InputIterator, class _Tp>
_Tp accumulate(_InputIterator __first, _InputIterator __last, _Tp __init)
{
  __STL_REQUIRES(_InputIterator, _InputIterator);
  for ( ; __first != __last; ++__first)
    __init = __init + *__first;  // 累加到初始值
  return __init;
}

template <class _InputIterator, class _Tp, class _BinaryOperation>
_Tp accumulate(_InputIterator __first, _InputIterator __last, _Tp __init,
               _BinaryOperation __binary_op)
{
  __STL_REQUIRES(_InputIterator, _InputIterator);
  for ( ; __first != __last; ++__first)
    __init = __binary_op(__init, *__first);  // 二元操作累加到初始值
  return __init;
}
```

## 6.3.3 adjacent_difference

给定一个区间，计算每对相邻元素之间的差，并返回该区间。计算差值的过程也可以泛化为二元操作。

```cpp
template <class _InputIterator, class _OutputIterator, class _Tp>
_OutputIterator
__adjacent_difference(_InputIterator __first, _InputIterator __last,
                      _OutputIterator __result, _Tp*)
{
  _Tp __value = *__first;  // value 特指前一个元素
  while (++__first != __last) {
    _Tp __tmp = *__first;  // tmp 特指后一个元素
    *++__result = __tmp - __value;  // 计算差值
    __value = __tmp;       // 后一个元素成为前一个元素
  }
  return ++__result;  // 返回输出区间的尾迭代器
}

template <class _InputIterator, class _OutputIterator>
_OutputIterator
adjacent_difference(_InputIterator __first,
                    _InputIterator __last, _OutputIterator __result)
{
  __STL_REQUIRES(_InputIterator, _InputIterator);
  __STL_REQUIRES(_OutputIterator, _OutputIterator);
  if (__first == __last) return __result;
  *__result = *__first;  // 第一个元素之前没有元素，差值就是它本身
  return __adjacent_difference(__first, __last, __result,
                               __VALUE_TYPE(__first));  // 取迭代器的数据类型
}

template <class _InputIterator, class _OutputIterator, class _Tp,
          class _BinaryOperation>
_OutputIterator
__adjacent_difference(_InputIterator __first, _InputIterator __last,
                      _OutputIterator __result, _Tp*,
                      _BinaryOperation __binary_op) {
  _Tp __value = *__first;  // value 特指前一个元素
  while (++__first != __last) {
    _Tp __tmp = *__first;  // tmp 特指后一个元素
    *++__result = __binary_op(__tmp, __value);  // 泛化计算差值
    __value = __tmp;       // 后一个元素成为前一个元素
  }
  return ++__result;  // 返回输出区间的尾迭代器
}

template <class _InputIterator, class _OutputIterator, class _BinaryOperation>
_OutputIterator
adjacent_difference(_InputIterator __first, _InputIterator __last,
                    _OutputIterator __result, _BinaryOperation __binary_op)
{
  __STL_REQUIRES(_InputIterator, _InputIterator);
  __STL_REQUIRES(_OutputIterator, _OutputIterator);
  if (__first == __last) return __result;
  *__result = *__first;  // 第一个元素之前没有元素，差值就是它本身
  return __adjacent_difference(__first, __last, __result,
                               __VALUE_TYPE(__first),
                               __binary_op);
}
```

如果参数 `result` 与 `first` 相同，那么就是一个 **就地操作**。

## 6.3.4 inner_product

计算两个序列的内积。内积的定义是两个序列的对应元素相乘，然后累加到一个值上。泛化版本可以提供两个二元操作，分别泛化相乘和累加两个过程。

```cpp
template <class _InputIterator1, class _InputIterator2, class _Tp>
_Tp inner_product(_InputIterator1 __first1, _InputIterator1 __last1,
                  _InputIterator2 __first2, _Tp __init)
{
  __STL_REQUIRES(_InputIterator2, _InputIterator);
  __STL_REQUIRES(_InputIterator2, _InputIterator);
  for ( ; __first1 != __last1; ++__first1, ++__first2)
    __init = __init + (*__first1 * *__first2);  // 相乘并累加
  return __init;
}

template <class _InputIterator1, class _InputIterator2, class _Tp,
          class _BinaryOperation1, class _BinaryOperation2>
_Tp inner_product(_InputIterator1 __first1, _InputIterator1 __last1,
                  _InputIterator2 __first2, _Tp __init,
                  _BinaryOperation1 __binary_op1,
                  _BinaryOperation2 __binary_op2)
{
  __STL_REQUIRES(_InputIterator2, _InputIterator);
  __STL_REQUIRES(_InputIterator2, _InputIterator);
  for ( ; __first1 != __last1; ++__first1, ++__first2)
    __init = __binary_op1(__init, __binary_op2(*__first1, *__first2));  // op2 泛化相乘，op1 泛化累加
  return __init;
}
```

## 6.3.5 partial_sum

部分和 (也可以称为是 **前缀和**)。给定一个区间，计算每个位置之前的所有元素之和。同样，求和操作可以泛化为二元操作。

```cpp
template <class _InputIterator, class _OutputIterator, class _Tp>
_OutputIterator
__partial_sum(_InputIterator __first, _InputIterator __last,
              _OutputIterator __result, _Tp*)
{
  _Tp __value = *__first;  // 累加值
  while (++__first != __last) {
    __value = __value + *__first;  // 累加
    *++__result = __value;         // 赋值到目标位置
  }
  return ++__result;       // 返回目标位置的尾迭代器
}

template <class _InputIterator, class _OutputIterator>
_OutputIterator
partial_sum(_InputIterator __first, _InputIterator __last,
            _OutputIterator __result)
{
  __STL_REQUIRES(_InputIterator, _InputIterator);
  __STL_REQUIRES(_OutputIterator, _OutputIterator);
  if (__first == __last) return __result;
  *__result = *__first;
  return __partial_sum(__first, __last, __result, __VALUE_TYPE(__first));
}

template <class _InputIterator, class _OutputIterator, class _Tp,
          class _BinaryOperation>
_OutputIterator
__partial_sum(_InputIterator __first, _InputIterator __last,
              _OutputIterator __result, _Tp*, _BinaryOperation __binary_op)
{
  _Tp __value = *__first;
  while (++__first != __last) {
    __value = __binary_op(__value, *__first);  // 泛化累加为二元操作
    *++__result = __value;
  }
  return ++__result;
}

template <class _InputIterator, class _OutputIterator, class _BinaryOperation>
_OutputIterator
partial_sum(_InputIterator __first, _InputIterator __last,
            _OutputIterator __result, _BinaryOperation __binary_op)
{
  __STL_REQUIRES(_InputIterator, _InputIterator);
  __STL_REQUIRES(_OutputIterator, _OutputIterator);
  if (__first == __last) return __result;
  *__result = *__first;
  return __partial_sum(__first, __last, __result, __VALUE_TYPE(__first),
                       __binary_op);
}
```

## 6.3.6 power

该算法由 SGI 专属，不在 STL 标准中。用于计算某个数的 `n` 次幂，默认使用乘法计算乘幂。当然，也可以用泛化版的算法，用其它二元操作替换乘法，但二元操作 **必须满足结合律**。

```cpp
template <class _Tp, class _Integer, class _MonoidOperation>
_Tp __power(_Tp __x, _Integer __n, _MonoidOperation __opr)
{
  if (__n == 0)
    return identity_element(__opr);
  else {
    while ((__n & 1) == 0) {
      __n >>= 1;
      __x = __opr(__x, __x);
    }

    _Tp __result = __x;
    __n >>= 1;
    while (__n != 0) {
      __x = __opr(__x, __x);  // 快速幂？
      if ((__n & 1) != 0)
        __result = __opr(__result, __x);
      __n >>= 1;
    }
    return __result;
  }
}

template <class _Tp, class _Integer>
inline _Tp __power(_Tp __x, _Integer __n)
{
  return __power(__x, __n, multiplies<_Tp>());  // 默认使用乘法
}

// Alias for the internal name __power.  Note that power is an extension,
// not part of the C++ standard.

template <class _Tp, class _Integer, class _MonoidOperation>
inline _Tp power(_Tp __x, _Integer __n, _MonoidOperation __opr)
{
  return __power(__x, __n, __opr);
}

template <class _Tp, class _Integer>
inline _Tp power(_Tp __x, _Integer __n)
{
  return __power(__x, __n);
}
```

## 6.3.7 iota

SGI 专属，不在 STL 标准中。设置某个区间的内容，使得区间内元素从一个指定的 `value` 开始递增。

```cpp
// iota is not part of the C++ standard.  It is an extension.

template <class _ForwardIter, class _Tp>
void
iota(_ForwardIter __first, _ForwardIter __last, _Tp __value)
{
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  __STL_CONVERTIBLE(_Tp, typename iterator_traits<_ForwardIter>::value_type);
  while (__first != __last)
    *__first++ = __value++;  // 从第一个元素开始，从 value 开始递增
}
```
