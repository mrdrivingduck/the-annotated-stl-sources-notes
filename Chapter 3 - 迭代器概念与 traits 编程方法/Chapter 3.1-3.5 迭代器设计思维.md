# Chapter 3.1-3.5 - 迭代器设计思维

Created by : Mr Dk.

2021 / 03 / 31 18:44

Nanjing, Jiangsu, China

---

**迭代器 (iterator)** 模式是 *Design Patterns* 中提供的 23 个设计模式之一，定义为：提供一种方法，使之能够依序巡防某个聚合物 (容器) 所含的各种元素，而又无需暴露该聚合物的内部表述方式。

STL 的中心思想在于，将 **数据容器** 与 **算法** 分开，彼此独立设计，最后再以粘合剂将它们撮合在一起。容器的泛型化可以由 C++ 的 class templates 完成；算法的泛型化可以由 C++ 的 function templates 完成；迭代器实现容器和算法的粘合。

迭代器是一种行为类似指针的对象，而指针最常用的操作是：

* `operator*`
* `operator->`

因此，迭代器最重要的工作就是对以上两个运算符进行重载。

另外，在迭代器设计时，应当尽可能不暴露迭代对象的设计细节。因此，迭代器的设计工作应当交给相应数据结构的设计者。这样可以把所有实现细节封装起来不被使用者看到。STL 中每一种容器都有专属的迭代器。

## 3.3 迭代器的关联类型 (Associated Types)

迭代器既然功能类似指针，那么除了迭代器自身的数据类型 `I` 以外，还应该知道自身指向数据的数据类型 `T`，及其它的相关联的数据类型。利用 C++ 函数模板的参数推导功能，可以部分解决这样的问题：

```c++
template <class I, class T>
void func_impl(I iter, T t)
{
    T tmp; // 迭代器指向的类型
    // ...
}

template <class I>
void func(I iter)
{
    func_impl(iter, *iter);
}

int main()
{
    int i;
    func(&i);
}
```

通过将 `func()` 的所有逻辑都移到 `func_impl()` 中实现，然后利用类型推导，成功在 `func_impl()` 中获得了迭代器指向的类型 `T`。问题在于，用户最终调用的是 `func()`。如果希望 `func()` 返回迭代器指向的数据类型，那么 `func()` 如何能够返回迭代器指向的数据类型 `T` 呢？

通过在迭代器类内 **显式声明内嵌类型** 即可。比如将迭代器指向的数据类型统一命名为 `value_type`：

```c++
template <class T>
struct MyIter {
    typedef T value_type; // 将内嵌类型显式声明为 value_type 类型
    
    T *ptr;
    // ...
};

template <class I>
typename I::value_type // func 函数的返回值为 I 的 value_type 类型
func(I iter)
{
    return *iter;
}
```

但是 STL 必须接受 C++ 原生指针作为迭代器，而原生指针并没有 `value_type` 的定义。通过 C++ 模板的 **部分具体化**，可以为原生指针类型单独实现一套特殊版本的模板：

```c++
template <class T>
struct MyIter<T*> {
    // ...
};
```

## 3.4 Traits 编程方法 - STL 源代码的钥匙

STL 定义了一个专门用于 **萃取** 迭代器特性 (包含其内嵌类型) 的结构体。假设我们关注迭代器的特性只有内嵌类型 `value_type` (实际上还会有别的)，那么模板结构体的定义为：

```c++
template <class I>
struct iterator_traits {
    typedef typename I::value_type value_type;
};
```

如果 `I` 类型有自己的 `value_type` 定义，那么 `I` 的 `value_type` 类型就是该结构体的 `value_type` 类型。`func()` 函数可改写为：

```c++
template <class I>
typename iterator_traits<I>::value_type // func 的返回类型
func(I iter)
{
    return *iter;
}
```

对于原生指针类型，实现一个部分具体化的结构体模板，使其内嵌类型为原生指针指向的数据类型：

```c++
template <class T>
struct iterator_traits<T*> {
    typedef T value_type;
};
```

另外，还有常量指针。设计另一个部分具体化的版本即可：

```c++
template <class T>
struct iterator_traits<const T*> {
    typedef T value_type;
};
```

综上，`iterator_traits` 实现了类似 **榨汁机** 的功能，把迭代器相关联的数据类型给榨取出来。当然，STL 中所有的迭代器都要遵循约定，定义出迭代器所有的关联数据类型。不遵守约定的迭代器将无法兼容 STL。除了指向的数据类型 (`value_type`) 外，迭代器需要定义五种相关联的数据类型。定义如下：

```c++
template <class I>
struct iterator_traits {
    typedef typename I::iterator_category iterator_category;
    typedef typename I::value_type        value_type;
    typedef typename I::difference_type   difference_type;
    typedef typename I::pointer           pointer;
    typedef typename I::reference         reference;
};
```

对于原生指针、原生常量指针，需要对这个结构体模板进行部分具体化，正确表示这五个数据类型。

这些数据类型的含义如下。

### `value_type`

迭代器所指对象的数据类型，即 **内嵌类型**。

### `difference_type`

表示两个迭代器之间的距离。对于 C++ 原生指针和原生常量指针，需要部分具体化，使用 C++ 自带的 `ptrdiff_t` (定义在 `<sctddef>` 头文件中) 作为原生指针之间的距离：

```c++
template <class I>
struct iterator_traits<T*>
{
    typedef ptrdiff_t difference_type;
};

template <class I>
struct iterator_traits<const T*>
{
    typedef ptrdiff_t difference_type;
};
```

### `reference_type`

根据迭代器所指数据是否允许被改变，当函数需要返回 **左值** (允许被赋值) 时，应当以引用类型的方式返回。

### `pointer_type`

指向迭代器所指数据的指针类型。同样需要对原生指针和原生常量指针进行特化。

```c++
template <class T>
struct iterator_traits<T*>
{
    typedef T* pointer;
    typedef T& reference;
};

template <class T>
struct iterator_traits<const T*>
{
    typedef const T* pointer;
    typedef const T& reference;
};
```

### `iterator_category`

用于区分迭代器的操作类型。根据移动特性分类，迭代器可以被分为：

* Input Iterator：迭代器所指对象 **只读**
* Output Iterator：迭代器所指对象 **只写**
* Forward Iterator：迭代器所指区间范围内可读可写，但只能向前移动
* Bidirectional Iterator：迭代器在区间上可 **双向移动**
* Random Access Iterator：涵盖指针的所有算数能力，除了双向移动，还能 **跳跃移动** (随机访问)

这五种迭代器有着天然的从属关系：

```
Input Iterator  --
                  |--> Forward Iterator --> Bidirectional Iterator --> Random Access Iterator
Output Iterator --
```

显然，如果迭代器支持更高级的操作，那么使用低级的操作方式将使性能下降。比如，如果要使迭代器前进三个单位，对于 Forward Iterator 来说需要循环，而对 Random Access Iterator 来说只需要直接将指针 + 3 即可。对于算法库中的函数模板，需要根据迭代器的不同类型，分别实现不同性能的操作方式。如何判断迭代器的类型呢？使用 traits 中的 `iterator_category()` 属性作为函数的最后一个参数，使编译器根据迭代器类型选择相应的重载函数。

五种迭代器根据从属关系定义为五个结构体。结构体内没有任何成员，因为只作为标记使用：

```c++
struct input_iterator_tag {  };
struct output_iterator_tag {  };
struct forward_iterator_tag : public input_iterator_tag {  };
struct bidirectional_iterator_tag : public forward_iterator_tag {  };
struct random_access_iterator_tag : public bidirectional_iterator_tag {  };
```

比如，对于算法模板中的 `advance()` 来说，其入口被定义为：

```c++
template <class InputIterator, class Distance>
inline void advance(InputIterator& i, Distance n)
{
    __advance(i, n, iterator_traits<InputIterator>::iterator_category());
}
```

代码中产生了 `iterator_category` 对应的临时对象，根据对象类型，编译器选择合适的重载函数。重载的版本有以下几种，实现各不相同，并且最后一个参数只有类型没有变量名 (因为仅用于让编译器区分版本，具体实现内不需要使用)。

```c++
template <class InputIterator, class Distance>
inline void __advance(InputIterator& i, Distance n, input_iterator_tag)
{
    // input_iterator 版本
    while (n--) ++i;
}

template <class ForwardIterator, class Distance>
inline void __advance(ForwardIterator& i, Distance n, forward_iterator_tag)
{
    // forward_iterator 版本
    __advance(i, n, input_iterator_tag()); // 传递调用 input_iterator 版本
}

template <class BidirectionalIterator, class Distance>
inline void __advance(BidirectionalIterator& i, Distance n, bidirectional_iterator_tag)
{
    // bidirectional_iterator 版本
    if (n >= 0)
        while (n--) ++i;
    else
        while (n++) --i;
}

template <class RandomAccessIterator, class Distance>
inline void __advance(RandomAccessIterator& i, Distance n, random_access_iterator_tag)
{
    i += n;
}
```

当然，对于这个迭代器关联类型，也需要为原生指针和原生常量指针特化出一个版本。由于原生指针和原生常量指针都是随机访问的迭代器，因此直接将迭代器类型特化为 Random Access Iterator：

```c++
template <class T>
struct iterator_traits<T*> {
    typedef random_access_iterator_tag iterator_category;
};

template <class T>
struct iterator_traits<const T*> {
    typedef random_access_iterator_tag iterator_category;
};
```

另外，对于算法库中的函数模板，参数列表中的迭代器类型以它可以接受的 **最低级迭代器** 的类型为参数命名。比如 `advance()` 函数，能够接受 Input Iterator 以上的所有类型迭代器，因此该函数被定义为：

```c++
template <class InputIterator, class Distance>
inline void advance(InputIterator& i, Distance n)
{
    __advance(i, n, iterator_traits<InputIterator>::iterator_category());
}
```

也就是说，低于 Input Iterator 等级的迭代器 (如果有) 将无法作为 `advance()` 的参数。

## 3.5 `std::iterator` 的保证

如果一个迭代器不提供上述五个关联数据类型，traits 机制将无法工作，导致自别于整个 STL 架构，无法与 STL 其它组件顺利搭配。STL 提供了一个类，使得如果每个新设计的迭代器都能继承自它，就可以保证符合 STL 规范：

```c++
template <class _Category,
          class _Tp,
          class _Distance = ptrdiff_t,
          class _Pointer = _Tp*,
          class _Reference = _Tp&>
struct iterator {
  typedef _Category  iterator_category;
  typedef _Tp        value_type;
  typedef _Distance  difference_type;
  typedef _Pointer   pointer;
  typedef _Reference reference;
};
```

其中，后三个参数都有默认值，因此只需要提供前两个参数即可：

* 迭代器类型
* 迭代器的内嵌类型

---

