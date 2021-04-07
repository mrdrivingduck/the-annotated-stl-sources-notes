# Chapter 7 - 仿函数 functors

Created by : Mr Dk.

2021 / 04 / 07 20:44

Nanjing, Jiangsu, China

---

## 7.1 仿函数 (functors) 概观

仿函数 (functors) 是早期的命名方式。在新的 C++ 标准规格中，采用的新名称是 **函数对象 (function objects)**。其实际意义是：一种具有函数特质的对象，该对象可以像函数一样被调用。

仿函数在 STL 中主要作用于算法领域：

* 算法的第一个版本使用最常用 (或最直观) 的某种运算 (比如 `operator<`)
* 算法的第二个版本使用用户提供的仿函数操作，实现算法的 **泛化** (比如用户自行提供一个比较函数)

如果想要将某个 **操作** 当作算法的参数，那么有两种方法：

1. 将操作设计为一个函数，将函数指针作为参数
2. 将操作设计为一个仿函数类，产生一个仿函数类的对象，以该对象作为参数

其中，函数指针无法满足 STL 对抽象性的要求，也无法满足 STL 适配器的要求。

从实现角度来说，仿函数实际上就是一个 **行为类似函数** 的对象，因此仿函数类必须重载 `operator()`。这样仿函数类的对象之后加上 `()` 就可以调用仿函数类内定义的操作逻辑。除了显式产生对象外，更多的用法是产生一个 **匿名的临时对象**，比如 `greater<int>()(6, 4)`。

仿函数按操作数个数划分：一元 / 二元；按功能划分：算术运算 / 关系运算 / 逻辑运算。

## 7.2 可适配 (Adaptable) 的关键

STL 的仿函数必须定义自己的 **相关类型 (associative types)** (和迭代器类似)，这些相关类型能够被函数适配器取出，从而获得仿函数的信息。这些类型只是类型定义，只会在编译期内使用，对程序执行效率没有影响，也没有空间负担。

仿函数的相关类型包括 **函数参数类型** 和 **函数返回值类型**。为方便起见，STL 中已经定义好了一元仿函数 (一个参数，一个返回值) 和二元仿函数 (两个参数，一个返回值)。

### 7.2.1 unary_function

一元仿函数。

```c++
template <class _Arg, class _Result>
struct unary_function {
  typedef _Arg argument_type;   // 参数类型
  typedef _Result result_type;  // 返回值类型
};
```

任何仿函数如果继承自上述结构体，那么就可以通过模板取得参数类型和返回值类型。

### 7.2.2 binary_function

二元仿函数。

```c++
template <class _Arg1, class _Arg2, class _Result>
struct binary_function {
  typedef _Arg1 first_argument_type;   // 第一参数类型
  typedef _Arg2 second_argument_type;  // 第二参数类型
  typedef _Result result_type;         // 返回值类型
};
```

任何仿函数如果继承自上述结构体，就可以通过模板取得两个参数的类型和返回值类型。

## 7.3 算术类 (Arithmetic) 仿函数

加法仿函数 (二元运算)：

```c++
template <class _Tp>
struct plus : public binary_function<_Tp,_Tp,_Tp> { // 两个参数类型与返回值类型一致
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x + __y; }
};
```

减法仿函数 (二元运算)：

```c++
template <class _Tp>
struct minus : public binary_function<_Tp,_Tp,_Tp> { // 两个参数类型与返回值类型一致
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x - __y; }
};
```

乘法仿函数 (二元运算)：

```c++
template <class _Tp>
struct multiplies : public binary_function<_Tp,_Tp,_Tp> { // 两个参数类型与返回值类型一致
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x * __y; }
};
```

除法仿函数 (二元运算)：

```c++
template <class _Tp>
struct divides : public binary_function<_Tp,_Tp,_Tp> { // 两个参数类型与返回值类型一致
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x / __y; }
};
```

模运算仿函数 (二元运算)：

```c++
template <class _Tp>
struct modulus : public binary_function<_Tp,_Tp,_Tp> 
{
  _Tp operator()(const _Tp& __x, const _Tp& __y) const { return __x % __y; }
};
```

取反运算仿函数 (一元运算)：

```c++
template <class _Tp>
struct negate : public unary_function<_Tp,_Tp> // 参数类型与返回类型一致
{
  _Tp operator()(const _Tp& __x) const { return -__x; }
};
```

此外，还有两个不符合 C++ 标准的，表示同一元素的仿函数 (二元运算)：

* `T + 0 = T`
* `T * 1 = T`

```c++
// identity_element (not part of the C++ standard).

template <class _Tp> inline _Tp identity_element(plus<_Tp>) {
  return _Tp(0);
}
template <class _Tp> inline _Tp identity_element(multiplies<_Tp>) {
  return _Tp(1);
}
```

## 7.4 关系运算类 (Relational) 仿函数

全部都是二元运算。返回值类型全部为 `bool`。

相等仿函数 (二元运算)：

```c++
template <class _Tp>
struct equal_to : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x == __y; }
};
```

不相等仿函数 (二元运算)：

```c++
template <class _Tp>
struct not_equal_to : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x != __y; }
};
```

大于仿函数 (二元运算)：

```c++
template <class _Tp>
struct greater : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x > __y; }
};
```

大于等于仿函数 (二元运算)：

```c++
template <class _Tp>
struct greater_equal : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x >= __y; }
};
```

小于仿函数 (二元运算)：

```c++
template <class _Tp>
struct less : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x < __y; }
};
```

小于等于仿函数 (二元运算)：

```c++
template <class _Tp>
struct less_equal : public binary_function<_Tp,_Tp,bool> 
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x <= __y; }
};
```

## 7.5 逻辑运算类 (Logical) 仿函数

与、或、非运算。返回值全部为 `bool`。

与运算仿函数 (二元运算)：

```c++
template <class _Tp>
struct logical_and : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x && __y; }
};
```

或运算仿函数 (二元运算)：

```c++
template <class _Tp>
struct logical_or : public binary_function<_Tp,_Tp,bool>
{
  bool operator()(const _Tp& __x, const _Tp& __y) const { return __x || __y; }
};
```

非运算仿函数 (二元运算)：

```c++
template <class _Tp>
struct logical_not : public unary_function<_Tp,bool>
{
  bool operator()(const _Tp& __x) const { return !__x; }
};
```

## 7.6 相同 (identity)、选择 (select)、投影 (project) 仿函数

C++ 标准不涵盖这类仿函数，但各个实现版本都实现了这类仿函数作为内部使用。

表示相同元素的仿函数 (一元运算)：

```c++
// identity is an extensions: it is not part of the standard.

template <class _Tp>
struct _Identity : public unary_function<_Tp,_Tp> {
  const _Tp& operator()(const _Tp& __x) const { return __x; }
};

template <class _Tp> struct identity : public _Identity<_Tp> {};
```

选择仿函数，接收一个 pair，返回 pair 中的第一个元素或第二个元素 (一元运算)：

```c++
// select1st and select2nd are extensions: they are not part of the standard.

template <class _Pair>
struct _Select1st : public unary_function<_Pair, typename _Pair::first_type> {
  const typename _Pair::first_type& operator()(const _Pair& __x) const {
    return __x.first;
  }
};

template <class _Pair>
struct _Select2nd : public unary_function<_Pair, typename _Pair::second_type>
{
  const typename _Pair::second_type& operator()(const _Pair& __x) const {
    return __x.second;
  }
};

template <class _Pair> struct select1st : public _Select1st<_Pair> {};
template <class _Pair> struct select2nd : public _Select2nd<_Pair> {};
```

投影仿函数，接收两个参数，返回一个参数 (二元运算)：

```c++
// project1st and project2nd are extensions: they are not part of the standard
template <class _Arg1, class _Arg2>
struct _Project1st : public binary_function<_Arg1, _Arg2, _Arg1> {
  _Arg1 operator()(const _Arg1& __x, const _Arg2&) const { return __x; }
};

template <class _Arg1, class _Arg2>
struct _Project2nd : public binary_function<_Arg1, _Arg2, _Arg2> {
  _Arg2 operator()(const _Arg1&, const _Arg2& __y) const { return __y; }
};

template <class _Arg1, class _Arg2> 
struct project1st : public _Project1st<_Arg1, _Arg2> {};

template <class _Arg1, class _Arg2>
struct project2nd : public _Project2nd<_Arg1, _Arg2> {};
```

---

