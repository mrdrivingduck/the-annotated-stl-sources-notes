# Chapter 6.7.7 - random_shuffle

Created by : Mr Dk.

2021 / 04 / 15 16:35

Nanjing, Jiangsu, China

---

该算法将 `[first, last)` 中的元素随机重排，在 `N!` (`N = last - first`) 种可能中随机选出一种。注意，需要保证算法能够在 `N!` 种可能性上均匀分布。另外，算法可以使用自带的随机数生成器打乱序列，也可以使用用户提供的随机数仿函数，这个仿函数是通过 **引用** 的方式传递：因为随机数生成器一般拥有 **本地状态**，每次调用时会发生改变。使用值传递将丢失每次调用对本地状态的修改。

```c++
template <class _RandomAccessIter>
inline void random_shuffle(_RandomAccessIter __first,
                           _RandomAccessIter __last) {
  __STL_REQUIRES(_RandomAccessIter, _Mutable_RandomAccessIterator);
  if (__first == __last) return;
  for (_RandomAccessIter __i = __first + 1; __i != __last; ++__i)  // 依次对每个位置上的元素进行对换
    iter_swap(__i, __first + __random_number((__i - __first) + 1));  // 模运算计算对换位置
}

// Return a random number in the range [0, __n).  This function encapsulates
// whether we're using rand (part of the standard C library) or lrand48
// (not standard, but a much better choice whenever it's available).

template <class _Distance>
inline _Distance __random_number(_Distance __n) {
#ifdef __STL_NO_DRAND48
  return rand() % __n;
#else
  return lrand48() % __n;
#endif
}

template <class _RandomAccessIter, class _RandomNumberGenerator>
void random_shuffle(_RandomAccessIter __first, _RandomAccessIter __last,
                    _RandomNumberGenerator& __rand) {
  __STL_REQUIRES(_RandomAccessIter, _Mutable_RandomAccessIterator);
  if (__first == __last) return;
  for (_RandomAccessIter __i = __first + 1; __i != __last; ++__i)
    iter_swap(__i, __first + __rand((__i - __first) + 1));  // mod
}
```
