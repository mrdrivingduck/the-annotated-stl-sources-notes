# Chapter 4.7 - heap

Created by : Mr Dk.

2021 / 04 / 04 15:42 🍀

Nanjing, Jiangsu, China

---

## 4.7.1 heap 概述

heap 不属于 SQL 容器组件，而是一种组织数据的方式，用于实现 priority_queue。优先队列允许用于以任意顺序将元素 push 到容器内，但取出时一定从最高优先级的数据开始取出。

如果使用平衡二叉树来维护所有元素，代价有些高。因为优先队列并不要求元素整体有序，每次只需要取出最大的那个元素即可，不需要其它元素之间有序。二叉堆是一颗完全二叉树，满足效率和代价的折衷。完全二叉树的树内没有洞，因此可以直接使用线性数组来存储结点，并且通过数组下标的倍数关系方便地取得左右孩子结点和父结点。

这样一来，只需要：

- 一个数组
- 一组 heap 算法

就能够实现堆。堆可以被分为 max-heap 和 min-heap。STL 提供的是 max-heap。

## 4.7.2 heap 算法

### `push_heap` 算法

该算法默认新的元素已经被插入在底层结构的 `end()` 迭代器所在的位置上。为了让这个新元素满足堆的定义，需要对该结点执行一个 _上浮 (percolate up)_ 操作，直到该元素的父结点比该元素大。

```c++
template <class _RandomAccessIterator, class _Compare>
inline void
push_heap(_RandomAccessIterator __first, _RandomAccessIterator __last,
          _Compare __comp) // 头尾迭代器指示区间，其中最后一个元素是新插入的；comp 表示比较方式
{
  __STL_REQUIRES(_RandomAccessIterator, _Mutable_RandomAccessIterator);
  __push_heap_aux(__first, __last, __comp,
                  __DISTANCE_TYPE(__first), __VALUE_TYPE(__first));
}
```

```c++
template <class _RandomAccessIterator, class _Distance, class _Tp>
inline void
__push_heap_aux(_RandomAccessIterator __first,
                _RandomAccessIterator __last, _Distance*, _Tp*)
{
  __push_heap(__first, _Distance((__last - __first) - 1), _Distance(0),
              _Tp(*(__last - 1))); // 指明堆的有效区间，以及最后一个位置的元素
}
```

```c++
template <class _RandomAccessIterator, class _Distance, class _Tp>
void
__push_heap(_RandomAccessIterator __first,
            _Distance __holeIndex, _Distance __topIndex, _Tp __value)
{
  _Distance __parent = (__holeIndex - 1) / 2; // 最后一个元素的父结点
  while (__holeIndex > __topIndex && *(__first + __parent) < __value) { // 父结点 < 最后一个结点
    *(__first + __holeIndex) = *(__first + __parent); // 父结点下移
    __holeIndex = __parent;                           // 父结点空缺
    __parent = (__holeIndex - 1) / 2;                 // 指向父结点的父结点
  }
  *(__first + __holeIndex) = __value; // 新元素放入合适的结点
}
```

### `pop_heap` 算法

取走根结点，将空出的根结点位置下放 (percolate down) 直到叶子结点；然后将堆的最右下结点填到空出的位置，做一次上浮 (percolate up) 操作。

```c++
template <class _RandomAccessIterator, class _Compare>
inline void
pop_heap(_RandomAccessIterator __first,
         _RandomAccessIterator __last, _Compare __comp)
{
  __STL_REQUIRES(_RandomAccessIterator, _Mutable_RandomAccessIterator);
  __pop_heap_aux(__first, __last, __VALUE_TYPE(__first), __comp);
}
```

```c++
template <class _RandomAccessIterator, class _Tp>
inline void
__pop_heap_aux(_RandomAccessIterator __first, _RandomAccessIterator __last,
               _Tp*)
{
  __pop_heap(__first, __last - 1, __last - 1,
             _Tp(*(__last - 1)), __DISTANCE_TYPE(__first));
}
```

```c++
template <class _RandomAccessIterator, class _Tp, class _Distance>
inline void
__pop_heap(_RandomAccessIterator __first, _RandomAccessIterator __last,
           _RandomAccessIterator __result, _Tp __value, _Distance*)
{
  *__result = *__first; // 将堆顶元素 (最大元素) 保存到空间的最后
  __adjust_heap(__first, _Distance(0), _Distance(__last - __first), __value);
}
```

```c++
template <class _RandomAccessIterator, class _Distance, class _Tp>
void
__adjust_heap(_RandomAccessIterator __first, _Distance __holeIndex,
              _Distance __len, _Tp __value)
{
  _Distance __topIndex = __holeIndex; // 堆顶目前空缺
  _Distance __secondChild = 2 * __holeIndex + 2; // 右孩子
  while (__secondChild < __len) {
    if (*(__first + __secondChild) < *(__first + (__secondChild - 1)))
      __secondChild--;
    *(__first + __holeIndex) = *(__first + __secondChild); // 左孩子
    __holeIndex = __secondChild;
    __secondChild = 2 * (__secondChild + 1);
  }
  if (__secondChild == __len) {
    *(__first + __holeIndex) = *(__first + (__secondChild - 1));
    __holeIndex = __secondChild - 1;
  }
  __push_heap(__first, __holeIndex, __topIndex, __value); // 将堆内最后一个元素移到空位，并上浮
}
```

最终，堆内的最大元素被放到了底层元素的尾端 (未被取走)，同时堆的区间范围比原先少 1。

### `sort_heap` 算法

迭代地对整个 heap 进行 `pop_heap()` 操作，那么每次堆区间的最后就是堆内最大元素。每次操作范围缩小 1，最终得到的就是一个递增的序列。

```c++
template <class _RandomAccessIterator>
void sort_heap(_RandomAccessIterator __first, _RandomAccessIterator __last)
{
  __STL_REQUIRES(_RandomAccessIterator, _Mutable_RandomAccessIterator);
  __STL_REQUIRES(typename iterator_traits<_RandomAccessIterator>::value_type,
                 _LessThanComparable);
  while (__last - __first > 1)
    pop_heap(__first, __last--);
}

template <class _RandomAccessIterator, class _Compare>
void
sort_heap(_RandomAccessIterator __first,
          _RandomAccessIterator __last, _Compare __comp)
{
  __STL_REQUIRES(_RandomAccessIterator, _Mutable_RandomAccessIterator);
  while (__last - __first > 1)
    pop_heap(__first, __last--, __comp);
}
```

### `make_heap` 算法

该算法将一段现有数据 **堆化**。找到区间内最后一个元素的父结点。以该结点为堆的根，进行堆调整。然后不断调整每一个父结点，直到父结点成为整个堆的根。

```c++
template <class _RandomAccessIterator, class _Compare,
          class _Tp, class _Distance>
void
__make_heap(_RandomAccessIterator __first, _RandomAccessIterator __last,
            _Compare __comp, _Tp*, _Distance*)
{
  if (__last - __first < 2) return;
  _Distance __len = __last - __first;
  _Distance __parent = (__len - 2)/2;

  while (true) {
    __adjust_heap(__first, __parent, __len, _Tp(*(__first + __parent)),
                  __comp);
    if (__parent == 0) return;
    __parent--;
  }
}

template <class _RandomAccessIterator, class _Compare>
inline void
make_heap(_RandomAccessIterator __first,
          _RandomAccessIterator __last, _Compare __comp)
{
  __STL_REQUIRES(_RandomAccessIterator, _Mutable_RandomAccessIterator);
  __make_heap(__first, __last, __comp,
              __VALUE_TYPE(__first), __DISTANCE_TYPE(__first));
}
```

## 4.7.3 heap 没有迭代器

heap 不提供遍历功能，也不提供迭代器。
