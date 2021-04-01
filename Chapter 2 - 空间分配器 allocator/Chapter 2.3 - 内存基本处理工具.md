# Chapter 2.3 - 内存基本处理工具

Created by : Mr Dk.

2021 / 04 / 01 22:35

Nanjing, Jiangsu, China

---

* `uninitialized_copy()` - 对应于高层算法 `copy()`
* `uninitialized_fill()` - 对应于高层算法 `fill()`
* `uninitialized_fill_n()` - 对应于高层算法 `fill_n()`

## 2.3.1 `uninitialized_copy`

```c++
template <class _InputIter, class _ForwardIter>
inline _ForwardIter
  uninitialized_copy(_InputIter __first, _InputIter __last,
                     _ForwardIter __result)
{
  return __uninitialized_copy(__first, __last, __result,
                              __VALUE_TYPE(__result));
}
```

对于 `__first` 和 `__last` 区间内的已有元素，将它们中的每一个复制一份，放到 `__result` 开始的未初始化区间中。通过对待复制元素的数据类型进行萃取，可以选择调用元素的拷贝构造函数，或是直接使用低层内存操作函数 `memmove()`。

上述代码已经使用 `iterator_traits` 萃取出了待复制元素的数据类型。接下来再使用 `__type_traits` 萃取出该数据类型是否为 POD (Plain Old Data) 类型 (C++ 原生数据类型及结构体)：

```c++
template <class _InputIter, class _ForwardIter, class _Tp>
inline _ForwardIter
__uninitialized_copy(_InputIter __first, _InputIter __last,
                     _ForwardIter __result, _Tp*)
{
  typedef typename __type_traits<_Tp>::is_POD_type _Is_POD;
  return __uninitialized_copy_aux(__first, __last, __result, _Is_POD());
}
```

对于是否为 POD，进一步具体化出了两种版本：

* 如果是 POD 类型，则调用 `copy()`
* 如果不是 POD 类型，则遍历每一个元素，依次调用拷贝构造函数

```c++
// Valid if copy construction is equivalent to assignment, and if the
//  destructor is trivial.
template <class _InputIter, class _ForwardIter>
inline _ForwardIter 
__uninitialized_copy_aux(_InputIter __first, _InputIter __last,
                         _ForwardIter __result,
                         __true_type)
{
  return copy(__first, __last, __result);
}

template <class _InputIter, class _ForwardIter>
_ForwardIter 
__uninitialized_copy_aux(_InputIter __first, _InputIter __last,
                         _ForwardIter __result,
                         __false_type)
{
  _ForwardIter __cur = __result;
  __STL_TRY {
    for ( ; __first != __last; ++__first, ++__cur)
      _Construct(&*__cur, *__first);
    return __cur;
  }
  __STL_UNWIND(_Destroy(__result, __cur));
}
```

而 `copy()` 中直接使用了 `memmove()` 进行高效拷贝：

```c++
static _Tp* copy(const _Tp* __first, const _Tp* __last, _Tp* __result) {
    const ptrdiff_t _Num = __last - __first;
    memmove(__result - _Num, __first, sizeof(_Tp) * _Num);
    return __result - _Num;
}
```

## 2.3.2 `uninitialized_fill`

```c++
template <class _ForwardIter, class _Tp>
inline void uninitialized_fill(_ForwardIter __first,
                               _ForwardIter __last, 
                               const _Tp& __x)
{
  __uninitialized_fill(__first, __last, __x, __VALUE_TYPE(__first));
}
```

对 `__first` 和 `__last` 之间的未初始化区间全部初始化为 `__x`。类似地，首先使用 `iterator_traits` 萃取出元素的数据类型。接下来，使用 `__type_traits` 萃取出数据类型是否为 POD 类型：

```c++
template <class _ForwardIter, class _Tp, class _Tp1>
inline void __uninitialized_fill(_ForwardIter __first, 
                                 _ForwardIter __last, const _Tp& __x, _Tp1*)
{
  typedef typename __type_traits<_Tp1>::is_POD_type _Is_POD;
  __uninitialized_fill_aux(__first, __last, __x, _Is_POD());
                   
}
```

如果不是 POD 类型，则依次循环每个元素进行拷贝构造；如果是 POD 类型，则直接调用 `fill()`：

```c++
// Valid if copy construction is equivalent to assignment, and if the
// destructor is trivial.
template <class _ForwardIter, class _Tp>
inline void
__uninitialized_fill_aux(_ForwardIter __first, _ForwardIter __last, 
                         const _Tp& __x, __true_type)
{
  fill(__first, __last, __x);
}

template <class _ForwardIter, class _Tp>
void
__uninitialized_fill_aux(_ForwardIter __first, _ForwardIter __last, 
                         const _Tp& __x, __false_type)
{
  _ForwardIter __cur = __first;
  __STL_TRY {
    for ( ; __cur != __last; ++__cur)
      _Construct(&*__cur, __x);
  }
  __STL_UNWIND(_Destroy(__first, __cur));
}
```

其中，`fill()` 直接使用指针进行循环复制：

```c++
template <class _ForwardIter, class _Tp>
void fill(_ForwardIter __first, _ForwardIter __last, const _Tp& __value) {
  __STL_REQUIRES(_ForwardIter, _Mutable_ForwardIterator);
  for ( ; __first != __last; ++__first)
    *__first = __value;
}
```

## 2.3.3 `uninitialized_fill_n`

```c++
template <class _ForwardIter, class _Size, class _Tp>
inline _ForwardIter 
uninitialized_fill_n(_ForwardIter __first, _Size __n, const _Tp& __x)
{
  return __uninitialized_fill_n(__first, __n, __x, __VALUE_TYPE(__first));
}
```

将 `__first` 开始的 `__n` 个内存空间的初始值全部设置为 `__x`。同样，先使用 `iterator_traits` 萃取出元素的数据类型，然后使用 `__type_traits` 萃取出数据类型是否为 POD：

```c++
template <class _ForwardIter, class _Size, class _Tp, class _Tp1>
inline _ForwardIter 
__uninitialized_fill_n(_ForwardIter __first, _Size __n, const _Tp& __x, _Tp1*)
{
  typedef typename __type_traits<_Tp1>::is_POD_type _Is_POD;
  return __uninitialized_fill_n_aux(__first, __n, __x, _Is_POD());
}
```

根据是否为 POD 类型，进一步具体化：

```c++
// Valid if copy construction is equivalent to assignment, and if the
//  destructor is trivial.
template <class _ForwardIter, class _Size, class _Tp>
inline _ForwardIter
__uninitialized_fill_n_aux(_ForwardIter __first, _Size __n,
                           const _Tp& __x, __true_type)
{
  return fill_n(__first, __n, __x);
}

template <class _ForwardIter, class _Size, class _Tp>
_ForwardIter
__uninitialized_fill_n_aux(_ForwardIter __first, _Size __n,
                           const _Tp& __x, __false_type)
{
  _ForwardIter __cur = __first;
  __STL_TRY {
    for ( ; __n > 0; --__n, ++__cur)
      _Construct(&*__cur, __x);
    return __cur;
  }
  __STL_UNWIND(_Destroy(__first, __cur));
}
```

其中，`fill_n()` 也是使用指针进行循环内存复制：

```c++
template <class _OutputIter, class _Size, class _Tp>
_OutputIter fill_n(_OutputIter __first, _Size __n, const _Tp& __value) {
  __STL_REQUIRES(_OutputIter, _OutputIterator);
  for ( ; __n > 0; --__n, ++__first)
    *__first = __value;
  return __first;
}
```

---

