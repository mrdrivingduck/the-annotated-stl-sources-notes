# Chapter 2.1 - SGI 空间分配器

Created by : Mr Dk.

2021 / 03 / 31 22:35

Nanjing, Jiangsu, China

---

空间分配器隐藏在 STL 的一切组件中，默默工作。为什么不把 allocator 称为 **内存分配器** 而是 **空间分配器** 呢？因为空间不一定是内存，也可以是磁盘或其它的存储介质。完全可以自行实现一个向磁盘索取空间的 allocator。但是在 SGI STL 中，空间的分配对象是 **内存**。

## 2.1 空间分配器的标准接口

- `rebind()`：？
- `allocator()` / `allocator(const allocator&)`：构造函数 / 拷贝构造函数
- `template <class U> allocator::allocator(const allocator<U>&)`：泛化的拷贝构造函数
- `~allocator()`：析构函数
- `address(reference x)` / `address(const_reference x)`：返回对象地址
- `allocate(size_type n, const void* = 0)`：分配存储 `n` 个对象的空间
- `deallocate(pointer p, size_type n)`：归还先前分配的空间
- `max_size()`：返回可成功分配的最大量
- `construct(pointer p, const T& x)`：构造对象
- `destroy(pointer p)`：析构对象

```cpp
template <class _Tp>
class allocator {
  typedef alloc _Alloc;          // The underlying allocator.
public:
  typedef size_t     size_type;
  typedef ptrdiff_t  difference_type;
  typedef _Tp*       pointer;
  typedef const _Tp* const_pointer;
  typedef _Tp&       reference;
  typedef const _Tp& const_reference;
  typedef _Tp        value_type;

  template <class _Tp1> struct rebind {
    typedef allocator<_Tp1> other;
  };

  allocator() __STL_NOTHROW {}
  allocator(const allocator&) __STL_NOTHROW {}
  template <class _Tp1> allocator(const allocator<_Tp1>&) __STL_NOTHROW {}
  ~allocator() __STL_NOTHROW {}

  pointer address(reference __x) const { return &__x; }
  const_pointer address(const_reference __x) const { return &__x; }

  // __n is permitted to be 0.  The C++ standard says nothing about what
  // the return value is when __n == 0.
  _Tp* allocate(size_type __n, const void* = 0) {
    return __n != 0 ? static_cast<_Tp*>(_Alloc::allocate(__n * sizeof(_Tp)))
                    : 0;
  }

  // __p is not permitted to be a null pointer.
  void deallocate(pointer __p, size_type __n)
    { _Alloc::deallocate(__p, __n * sizeof(_Tp)); }

  size_type max_size() const __STL_NOTHROW
    { return size_t(-1) / sizeof(_Tp); }

  void construct(pointer __p, const _Tp& __val) { new(__p) _Tp(__val); }
  void destroy(pointer __p) { __p->~_Tp(); }
};
```

SGI STL 在空间分配器上脱离了 STL 的标准规格，自行实现了一个专属的、拥有层次分配能力的、效率优越的特殊分配器。但是实际上 SGI STL 仍然提供了一个标准的分配器接口 `simple_alloc`，对 SGI STL 自行实现的内存分配器做了一层隐藏：

```cpp
template<class _Tp, class _Alloc>
class simple_alloc {

public:
    static _Tp* allocate(size_t __n)
      { return 0 == __n ? 0 : (_Tp*) _Alloc::allocate(__n * sizeof (_Tp)); }
    static _Tp* allocate(void)
      { return (_Tp*) _Alloc::allocate(sizeof (_Tp)); }
    static void deallocate(_Tp* __p, size_t __n)
      { if (0 != __n) _Alloc::deallocate(__p, __n * sizeof (_Tp)); }
    static void deallocate(_Tp* __p)
      { _Alloc::deallocate(__p, sizeof (_Tp)); }
};
```

所有的 SGI STL 容器全部使用这个 `simple_alloc` 接口。

SGI STL 使用了自行实现的空间分配器 `alloc`，与标准规范中的 `allocator` 不同。

```cpp
// 标准写法
vector<int, std::allocator<int> > vec;

// SGI STL in GCC
vector<int, std::alloc> vec;
```

不过 SGI STL 在实现所有容器时，已经将默认的空间分配器设置为 SGI 自行实现的 `alloc`：

```cpp
template <class T, class Alloc = alloc>
class vector { ... };
```

所以对于代码移植应该也没有什么问题，使用默认的缺省空间分配器即可。

## 2.2 SGI 标准的空间分配器 `std::allocator`

SGI 中定义了一个符合标准且名为 `allocator` 的分配器，但是 SGI 自己不使用它，也不建议开发者使用，因为 **效率不佳**。其本质只是对 C++ 原生的 `operator new` 和 `operator delete` 做了一层封装，并没有考虑到效率上的强化。

```cpp
template <class T>
inline T* allocate(ptrdiff_t size, T*) {
    set_new_handler(0);
    T* tmp = (T*)(::operator new((size_t)(size * sizeof(T))));
    if (tmp == 0) {
	cerr << "out of memory" << endl;
	exit(1);
    }
    return tmp;
}


template <class T>
inline void deallocate(T* buffer) {
    ::operator delete(buffer);
}

template <class T>
class allocator {
public:
    typedef T value_type;
    typedef T* pointer;
    typedef const T* const_pointer;
    typedef T& reference;
    typedef const T& const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    pointer allocate(size_type n) {
	return ::allocate((difference_type)n, (pointer)0);
    }
    void deallocate(pointer p) { ::deallocate(p); }
    pointer address(reference x) { return (pointer)&x; }
    const_pointer const_address(const_reference x) {
	return (const_pointer)&x;
    }
    size_type init_page_size() {
	return max(size_type(1), size_type(4096/sizeof(T)));
    }
    size_type max_size() const {
	return max(size_type(1), size_type(UINT_MAX/sizeof(T)));
    }
};
```
