# Chapter 4.2 - vector

Created by : Mr Dk.

2021 / 04 / 03 20:55

Nanjing, Jiangsu, China

---

## 4.2.1 概述

C/C++ 原生的数组是静态空间，一旦配置了就不能改变。vector 使用动态空间，随着元素的加入，由内部机制自行扩充空间以容纳元素。因此对内存的合理利用与灵活性有很大帮助，**吃多少用多少**。其实现技术的关键在于：

1. 内存大小控制
2. 内存重新分配时数据移动的效率

## 4.2.3 vector 的迭代器

vector 维护的是 **连续线性空间**，因此普通指针可以满足作为 vector 迭代器的所有条件。vector 支持 **随机存取**，因此提供 Random Access Iterator。

```c++
template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
class vector : protected _Vector_base<_Tp, _Alloc> 
{
public:
  typedef value_type* iterator;
  // ...
};
```

## 4.2.4 vector 的数据结构

由于 vector 使用连续的线性空间，因此使用了 `start` 和 `finish` 两个迭代器指示分配到的空间中正在使用的区间，`end_of_storage` 迭代器指示整块连续空间 (包含备用空间) 的尾端。

```c++
template <class _Tp, class _Allocator>
class _Vector_alloc_base<_Tp, _Allocator, true> {
protected:
  _Tp* _M_start;
  _Tp* _M_finish;
  _Tp* _M_end_of_storage;
};
```

为降低重新分配内存时的时间成本，vector 实际分配的内存大小会比用户需求量更大一些，以备将来可能的扩充。实际分配的内存量被称为 **容量 (capacity)**。根据上述三个迭代器，有许多成员函数已经可以被实现：

```c++
template <class _Tp, class _Alloc = __STL_DEFAULT_ALLOCATOR(_Tp) >
class vector : protected _Vector_base<_Tp, _Alloc> 
{
public:
  iterator begin() { return _M_start; }
  const_iterator begin() const { return _M_start; }
  iterator end() { return _M_finish; }
  const_iterator end() const { return _M_finish; }
    
  size_type size() const
    { return size_type(end() - begin()); }
  size_type max_size() const
    { return size_type(-1) / sizeof(_Tp); }
  size_type capacity() const
    { return size_type(_M_end_of_storage - begin()); }
  bool empty() const
    { return begin() == end(); }
    
  reference operator[](size_type __n) { return *(begin() + __n); }
  const_reference operator[](size_type __n) const { return *(begin() + __n); }
    
  reference front() { return *begin(); }
  const_reference front() const { return *begin(); }
  reference back() { return *(end() - 1); }
  const_reference back() const { return *(end() - 1); }
};
```

## 4.2.5 vector 的构造与内存管理：constructor，push_back

vector 使用空间分配器分配内存，并在类内定义了一个分配器，用于 **以元素大小为单位** 分配内存。

```c++
template <class _Tp, class _Alloc> 
class _Vector_base {
protected:
  typedef simple_alloc<_Tp, _Alloc> _M_data_allocator;
  _Tp* _M_allocate(size_t __n)
    { return _M_data_allocator::allocate(__n); }
  void _M_deallocate(_Tp* __p, size_t __n) 
    { _M_data_allocator::deallocate(__p, __n); }
};
```

vector 提供多个构造函数，允许用户指定空间大小以及初值。

```c++
_Vector_base(const _Alloc&)
    : _M_start(0), _M_finish(0), _M_end_of_storage(0) {}
_Vector_base(size_t __n, const _Alloc&)
    : _M_start(0), _M_finish(0), _M_end_of_storage(0) 
    {
        _M_start = _M_allocate(__n);
        _M_finish = _M_start;
        _M_end_of_storage = _M_start + __n;
    }
```

```c++
explicit vector(const allocator_type& __a = allocator_type())
    : _Base(__a) {}

vector(size_type __n, const _Tp& __value,
       const allocator_type& __a = allocator_type()) 
    : _Base(__n, __a)
    { _M_finish = uninitialized_fill_n(_M_start, __n, __value); }

explicit vector(size_type __n)
    : _Base(__n, allocator_type())
    { _M_finish = uninitialized_fill_n(_M_start, __n, _Tp()); }

vector(const vector<_Tp, _Alloc>& __x) 
    : _Base(__x.size(), __x.get_allocator())
    { _M_finish = uninitialized_copy(__x.begin(), __x.end(), _M_start); }
```

其中，`uninitialized_fill_n()` 和 `uninitialized_copy()` 会根据元素的数据类型自行选择效率较高的赋值或拷贝方式。

调用 `push_back()` 将新元素插入到 vector 尾端时，首先需要检查备用空间：

* 如果有备用空间，那么直接构造元素并调整 `finish` 迭代器
* 如果没有备用空间，那么需要扩容 (重新分配内存，移动数据，释放原内存)

```c++
void push_back(const _Tp& __x) {
    if (_M_finish != _M_end_of_storage) {
        construct(_M_finish, __x);
        ++_M_finish;
    }
    else
        _M_insert_aux(end(), __x);
}
```

内存重新分配的原则：

* 如果原大小为 0，那么分配 1 个元素的空间
* 如果原大小不为 0，那么分配原大小两倍的空间

使用 `uninitialized_copy()` 将原 vector 的内容拷贝到新 vector，并在结尾构造新元素。析构释放原 vector 的空间，并将三个迭代器指向新空间的正确位置。

```c++
template <class _Tp, class _Alloc>
void 
vector<_Tp, _Alloc>::_M_insert_aux(iterator __position, const _Tp& __x)
{
  if (_M_finish != _M_end_of_storage) {
    construct(_M_finish, *(_M_finish - 1));
    ++_M_finish;
    _Tp __x_copy = __x;
    copy_backward(__position, _M_finish - 2, _M_finish - 1);
    *__position = __x_copy;
  }
  else {
    const size_type __old_size = size();
    const size_type __len = __old_size != 0 ? 2 * __old_size : 1;
    iterator __new_start = _M_allocate(__len);
    iterator __new_finish = __new_start;
    __STL_TRY {
      __new_finish = uninitialized_copy(_M_start, __position, __new_start);
      construct(__new_finish, __x);
      ++__new_finish;
      __new_finish = uninitialized_copy(__position, _M_finish, __new_finish);
    }
    __STL_UNWIND((destroy(__new_start,__new_finish), 
                  _M_deallocate(__new_start,__len)));
    destroy(begin(), end());
    _M_deallocate(_M_start, _M_end_of_storage - _M_start);
    _M_start = __new_start;
    _M_finish = __new_finish;
    _M_end_of_storage = __new_start + __len;
  }
}
```

> 由于动态扩容并不是 **在原空间后接续新空间**，而是 **另外分配一块较大空间**，因此对 vector 的任何操作如果引起空间重新分配，那么指向原 vector 的迭代器都会 **失效**。

## 4.2.6 vector 的元素操作：pop_back，erase，clear，insert

`pop_back()` 的实现很简单。调整 `finish` 迭代器后，销毁最后一个元素即可。

```c++
void pop_back() {
    --_M_finish;
    destroy(_M_finish);
}
```

`erase()` 删除某个区间上的所有元素。通过调用全局的 `copy()` 函数，将被删除区间之后的所有元素复制到删除位置开始的位置，并把之后多余的元素给析构，最终重新设置 `finish` 迭代器。

```c++
iterator erase(iterator __position) {
    if (__position + 1 != end())
        copy(__position + 1, _M_finish, __position);
    --_M_finish;
    destroy(_M_finish);
    return __position;
}
iterator erase(iterator __first, iterator __last) {
    iterator __i = copy(__last, _M_finish, __first);
    destroy(__i, _M_finish);
    _M_finish = _M_finish - (__last - __first);
    return __first;
}
```

`clear()` 相当于对 `start` 和 `finish` 迭代器之间的区间做 `erase()`：

```c++
void clear() { erase(begin(), end()); }
```

`insert()` 函数分为两种情况：

* 备用空间大于等于新增元素个数，不需要触发扩容
* 备用空间长度小于新增元素个数，需要扩容并搬运

对于备用空间大于等于新增元素个数的场景，需要根据新增元素的个数是否多于插入位置之后的元素个数，合理调用 `uninitialized_copy()` / `copy()` (需要初始化 / 不需要初始化)，以追求性能。

对于备用空间不够的场景，既然新分配了内存，则按区间调用 `uninitialized_copy()` (复制 + 初始化) 即可。新区间的长度为原区间的两倍或原区间长度 + 新增元素个数的较大者。最终需要销毁并释放原空间，并调整迭代器指向新空间。

```c++
template <class _Tp, class _Alloc>
void 
vector<_Tp, _Alloc>::insert(iterator __position, 
                            const_iterator __first, 
                            const_iterator __last)
{
  if (__first != __last) {
    size_type __n = 0;
    distance(__first, __last, __n); // 待插入的元素个数 n
    if (size_type(_M_end_of_storage - _M_finish) >= __n) { // 备用空间够用
      const size_type __elems_after = _M_finish - __position; // 插入位置之后的元素个数
      iterator __old_finish = _M_finish;
      if (__elems_after > __n) { // 插入位置后的元素个数 > 待插入的元素个数
        uninitialized_copy(_M_finish - __n, _M_finish, _M_finish); // finish 迭代器之后 n 个元素初始化
        _M_finish += __n; // 将 finish 迭代器后移 n 个
        copy_backward(__position, __old_finish - __n, __old_finish); // 将剩余元素后移 (不用初始化)
        copy(__first, __last, __position); // 插入元素
      }
      else { // 插入位置后的元素个数 < 待插入的元素个数
        uninitialized_copy(__first + __elems_after, __last, _M_finish); // 复制并初始化待插入元素的后区间
        _M_finish += __n - __elems_after;
        uninitialized_copy(__position, __old_finish, _M_finish); // 复制并初始化插入位置后的元素
        _M_finish += __elems_after;
        copy(__first, __first + __elems_after, __position); // 复制待插入元素的前区间 (不用初始化)
      }
    }
    else { // 备用空间不足
      const size_type __old_size = size();
      const size_type __len = __old_size + max(__old_size, __n); // 新空间大小
      iterator __new_start = _M_allocate(__len); // 扩容
      iterator __new_finish = __new_start;
      __STL_TRY { // 按区间复制元素
        __new_finish = uninitialized_copy(_M_start, __position, __new_start);
        __new_finish = uninitialized_copy(__first, __last, __new_finish);
        __new_finish
          = uninitialized_copy(__position, _M_finish, __new_finish);
      }
      __STL_UNWIND((destroy(__new_start,__new_finish),
                    _M_deallocate(__new_start,__len))); // 失败回滚
      destroy(_M_start, _M_finish); // 销毁原区间
      _M_deallocate(_M_start, _M_end_of_storage - _M_start); // 释放原空间
      _M_start = __new_start; // 调整迭代器
      _M_finish = __new_finish;
      _M_end_of_storage = __new_start + __len;
    }
  }
}
```

---

