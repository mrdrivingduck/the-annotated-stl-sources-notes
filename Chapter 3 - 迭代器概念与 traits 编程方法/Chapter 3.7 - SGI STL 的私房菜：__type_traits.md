# Chapter 3.7 - SGI STL 的私房菜：\_\_type_traits

Created by : Mr Dk.

2021 / 03 / 31 20:40

Nanjing, Jiangsu, China

---

## About

在 STL 迭代器的实现中，traits 机制展露了很大的作用。通过 traits 技术，提取出了包括内嵌类型在内的五个迭代器关联数据类型。SGI 把这种方法扩展到了迭代器之外。`iterator_traits` 负责萃取迭代器的特性，而 `__type_traits` 则负责萃取数据类型的特性。数据类型有哪些特性值得被关注呢？

- 数据类型是否具有 non-trivial 的默认构造函数
- 数据类型是否具有 non-trivial 的拷贝构造函数
- 数据类型是否具有 non-trivial 的赋值运算符
- 数据类型是否具有 non-trivial 的析构函数
- 数据类型是否是 C++ 原生数据类型 (非对象类型)

> Trivial 可被翻译为简单的，不复杂的。一个简单的判断准则：如果类内包含指针类型的成员变量，并需要在构造函数或拷贝构造函数中对指针进行动态内存分配，在析构函数中对动态分配的内存进行回收，那么该构造函数就可被认为是 **non-trivial** 的。

萃取这些特性有什么用处？如果数据类型的构造函数是 trivial 的，那么在对该类型对象进行构造、析构、拷贝、赋值等操作时，可以采用 **更有效率的操作**，而不是构造函数。可以使用 `malloc()` / `memcpy()` 等内存操作获得最高效率。对于大规模且操作频繁的容器来说，可以显著提升效率。

\_\_type_traits 提供了一种机制，允许针对不同的数据类型，在编译时期完成函数的分派 (静态分派)。比如，在对一个未定类型的数组执行 copy 操作时，可以在编译期得知元素类型是否有 trivial 的拷贝构造函数，从而决定是否可以使用性能较高的 `memcpy()` / `memmove()`。

## Definition

由于编译器只会对对象形式的参数做参数推导，因此，定义两个结构体分别表示 `true` 和 `false`。结构体内是空的，因为这两个结构体仅用于标记真假 (是否有 non-trivial 的 XXX)。

```cpp
struct __true_type {
};

struct __false_type {
};
```

由此，`__type_traits` 内定义了五种数据类型特性：

```cpp
template <class _Tp>
struct __type_traits {
   typedef __true_type     this_dummy_member_must_be_first;
                   /* Do not remove this member. It informs a compiler which
                      automatically specializes __type_traits that this
                      __type_traits template is special. It just makes sure that
                      things work if an implementation is using a template
                      called __type_traits for something unrelated. */

   /* The following restrictions should be observed for the sake of
      compilers which automatically produce type specific specializations
      of this class:
          - You may reorder the members below if you wish
          - You may remove any of the members below if you wish
          - You must not rename members without making the corresponding
            name change in the compiler
          - Members you add will be treated like regular members unless
            you add the appropriate support in the compiler. */


   typedef __false_type    has_trivial_default_constructor;
   typedef __false_type    has_trivial_copy_constructor;
   typedef __false_type    has_trivial_assignment_operator;
   typedef __false_type    has_trivial_destructor;
   typedef __false_type    is_POD_type;
};
```

可以看到，对于所有的数据类型，其默认的五个特性都是 `__false_type`，即：

- 没有 trivial 的默认构造函数
- 没有 trivial 的拷贝构造函数
- 没有 trivial 的赋值运算符
- 没有 trivial 的析构函数
- 不是 C++ 原生的数据类型

显然，对于这种对象，只能老老实实调用该类型定义的构造函数 / 拷贝构造函数 / 析构函数 / 赋值运算符进行操作，而无法取得性能上的提升。换句话说，上述默认定义是对所有数据类型的 **保守值**。

另外，对于 C++ 所有的原生数据类型，可以对上述结构体模板进行部分具体化，修改模板默认的行为。这些数据类型的相关特性将是 trivial 的，因此可以使用 `memcpy()` 等最快速的方式来进行拷贝或赋值。

```cpp
__STL_TEMPLATE_NULL struct __type_traits<bool> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#endif /* __STL_NO_BOOL */

__STL_TEMPLATE_NULL struct __type_traits<char> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<signed char> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned char> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#ifdef __STL_HAS_WCHAR_T

__STL_TEMPLATE_NULL struct __type_traits<wchar_t> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#endif /* __STL_HAS_WCHAR_T */

__STL_TEMPLATE_NULL struct __type_traits<short> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned short> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<int> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned int> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<long> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned long> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#ifdef __STL_LONG_LONG

__STL_TEMPLATE_NULL struct __type_traits<long long> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<unsigned long long> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

#endif /* __STL_LONG_LONG */

__STL_TEMPLATE_NULL struct __type_traits<float> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<double> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};

__STL_TEMPLATE_NULL struct __type_traits<long double> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};
```

另外，C++ 原生的指针类型也被定义为是一种标量：

```cpp
template <class _Tp>
struct __type_traits<_Tp*> {
   typedef __true_type    has_trivial_default_constructor;
   typedef __true_type    has_trivial_copy_constructor;
   typedef __true_type    has_trivial_assignment_operator;
   typedef __true_type    has_trivial_destructor;
   typedef __true_type    is_POD_type;
};
```

## Application

以 STL 中的 `copy()` 函数模板为例。该函数包含多个特化版本和多个强化版本，都是为了效率考虑。其基本函数的入口定义如下：

```cpp
template <class T>
inline void copy(T* source, T* destination, int n)
{
    copy(source, destination, n, typename __type_traits<T>::has_trivial_copy_constructor());
}
```

在上述函数中，调用了重载版本的 `copy()` 函数，其最后一个参数将根据类型 `T` 的 `has_trivial_copy_constructor` 类型临时对象是 `__true_type` 对象还是 `__false_type` 对象，决定将函数调用分派到哪个版本的 `copy()` 上去：

```cpp
template <class T>
void copy(T* source, T* destination, int n, __false_type)
{
    // 拷贝构造函数为 non-trivial
    // 依次调用拷贝构造函数 (保守策略)
}

template <class T>
void copy(T* source, T* destination, int n, __true_type)
{
    // 拷贝构造函数为 trivial
    // 采用高效的方式实现拷贝
}
```

如果使用 SGI 的编译器，`__type_traits` 将有能力根据用户自定义类型是否有 trivial 的构造函数萃取出相应特性。但对于大部分缺乏这种功能的编译器来说，除 _POD (Plain Old Data)_ 本身以外的类型，`__type_traits` 只能萃取出 `__false_type`，从而使用保守策略。除非显式为自定义类型设计一个具体化的 `__type_traits` 版本，显式告诉编译器，该类型具有 trivial 的构造函数：

```cpp
template <> struct __type_traits<Shape> {
   typedef __true_type     has_trivial_default_constructor;
   typedef __false_type    has_trivial_copy_constructor;
   typedef __false_type    has_trivial_assignment_operator;
   typedef __false_type    has_trivial_destructor;
   typedef __false_type    is_POD_type;
};
```

综上，即使无法全面对自定义类型进行类型特性萃取，至少编译器还能够对原生 C++ (POD) 类型使用最高效的内存操作方式。因为它们全部都有具体化的 `__type_traits` 定义，且所有的 `typedef` 都是 `__true_type`。
