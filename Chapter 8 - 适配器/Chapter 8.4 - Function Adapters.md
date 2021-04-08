# Chapter 8.4 - Function Adapters

Created by : Mr Dk.

2021 / 04 / 08 18:05

Nanjing, Jiangsu, China

---

Function adapters 对仿函数进行适配。因此，每个 function adapters 类内都维护着一个被适配的仿函数对象成员。对该仿函数对象进行一些操作后，适配为功能略有不同的仿函数。

## 8.4.1 对返回值进行逻辑否定

对 (一元 / 二元) 仿函数的返回值进行逻辑非。

```c++
template <class _Predicate> // 模板参数为仿函数类
class unary_negate  // 一元仿函数的非运算还是一元仿函数，参数为仿函数的参数类型，返回值为 bool 类型
  : public unary_function<typename _Predicate::argument_type, bool> {
protected:
  _Predicate _M_pred;       // 类内维护一个一元仿函数对象
public:
  explicit unary_negate(const _Predicate& __x) : _M_pred(__x) {}
  bool operator()(const typename _Predicate::argument_type& __x) const {
    return !_M_pred(__x);  // 重载 operator()，将一元仿函数对象的运算结果取反并返回
  }
};

template <class _Predicate>
inline unary_negate<_Predicate> 
not1(const _Predicate& __pred)
{
  return unary_negate<_Predicate>(__pred);  // 返回一个被封装的一元仿函数
}
```

```c++
template <class _Predicate> 
class binary_negate  // 二元仿函数的非运算还是二元仿函数，参数为二元仿函数的两个参数类型，返回值为 bool 类型
  : public binary_function<typename _Predicate::first_argument_type,
                           typename _Predicate::second_argument_type,
                           bool> {
protected:
  _Predicate _M_pred;      // 类内维护一个二元仿函数对象
public:
  explicit binary_negate(const _Predicate& __x) : _M_pred(__x) {}
  bool operator()(const typename _Predicate::first_argument_type& __x, 
                  const typename _Predicate::second_argument_type& __y) const
  {
    return !_M_pred(__x, __y);  // 重载 operator()，将二元仿函数对象的运算结果取反并返回
  }
};

template <class _Predicate>
inline binary_negate<_Predicate> 
not2(const _Predicate& __pred)
{
  return binary_negate<_Predicate>(__pred);  // 返回一个被封装的二元仿函数
}
```

## 8.4.2 对参数进行绑定

在构造仿函数适配器时，传入二元仿函数对象和仿函数的其中一个参数。将该参数绑定为二元仿函数的第一个参数或第二个参数。

> 有啥用？假如想实现对任意一个数加 2 的仿函数，那么可以这样写：
>
> ```c++
> bind2nd(plus<int>() /* 二元仿函数对象 */, 2 /* 绑定为仿函数的第二参数 */);
> ```

```c++
template <class _Operation>  // 二元仿函数
class binder1st
  : public unary_function<typename _Operation::second_argument_type,  // 仿函数的第二参数类型
                          typename _Operation::result_type> {         // 仿函数的返回值类型
protected:
  _Operation op;  // 仿函数对象
  typename _Operation::first_argument_type value;  // 已绑定的仿函数第一参数的值
public:
  binder1st(const _Operation& __x,
            const typename _Operation::first_argument_type& __y)
      : op(__x), value(__y) {}  // 构造函数需要指定仿函数对象和要绑定的第一参数的值
  typename _Operation::result_type
  operator()(const typename _Operation::second_argument_type& __x) const {
    return op(value, __x);  // 以构造函数绑定的第一参数，以及参数指定的第二参数，调用仿函数
  }
};

template <class _Operation, class _Tp>
inline binder1st<_Operation> 
bind1st(const _Operation& __fn, const _Tp& __x) 
{
  typedef typename _Operation::first_argument_type _Arg1_type;
  return binder1st<_Operation>(__fn /* 仿函数对象 */, _Arg1_type(__x) /* 第一参数绑定值 */);
}
```

```c++
template <class _Operation>  // 二元仿函数
class binder2nd
  : public unary_function<typename _Operation::first_argument_type,  // 仿函数的第一参数类型
                          typename _Operation::result_type> {        // 仿函数的返回值类型
protected:
  _Operation op;  // 仿函数对象
  typename _Operation::second_argument_type value;  // 已绑定的仿函数第二参数的值
public:
  binder2nd(const _Operation& __x,
            const typename _Operation::second_argument_type& __y) 
      : op(__x), value(__y) {}  // 构造函数指定仿函数对象和要绑定的第二参数的值
  typename _Operation::result_type
  operator()(const typename _Operation::first_argument_type& __x) const {
    return op(__x, value);  // 以参数指定的第一参数，以及构造函数绑定的第二参数，调用仿函数
  }
};

template <class _Operation, class _Tp>
inline binder2nd<_Operation> 
bind2nd(const _Operation& __fn, const _Tp& __x) 
{
  typedef typename _Operation::second_argument_type _Arg2_type;
  return binder2nd<_Operation>(__fn /* 仿函数对象 */, _Arg2_type(__x) /* 第二参数绑定值 */);
}
```

## 8.4.3 用于函数合成

这里提到的两个适配器未纳入 STL 标准，是 SGI STL 的产物。

一元合成：`f1(f2(args))`

```c++
template <class _Operation1, class _Operation2>
class unary_compose
  : public unary_function<typename _Operation2::argument_type,
                          typename _Operation1::result_type> 
{
protected:
  _Operation1 _M_fn1;  // 仿函数 1
  _Operation2 _M_fn2;  // 仿函数 2
public:
  unary_compose(const _Operation1& __x, const _Operation2& __y) 
    : _M_fn1(__x), _M_fn2(__y) {}  // 构造函数
  typename _Operation1::result_type
  operator()(const typename _Operation2::argument_type& __x) const {
    return _M_fn1(_M_fn2(__x));    // 用参数调用仿函数 2，并将结果作为仿函数 1 的参数，调用仿函数 1
  }
};

template <class _Operation1, class _Operation2>
inline unary_compose<_Operation1,_Operation2> 
compose1(const _Operation1& __fn1, const _Operation2& __fn2)
{
  return unary_compose<_Operation1,_Operation2>(__fn1 /* 仿函数 1 */, __fn2 /* 仿函数 2 */);
}
```

二元合成：`f1(f2(args), f3(args))`

```c++
template <class _Operation1, class _Operation2, class _Operation3>
class binary_compose
  : public unary_function<typename _Operation2::argument_type,
                          typename _Operation1::result_type> {
protected:
  _Operation1 _M_fn1;  // 仿函数 1
  _Operation2 _M_fn2;  // 仿函数 2
  _Operation3 _M_fn3;  // 仿函数 3
public:
  binary_compose(const _Operation1& __x, const _Operation2& __y, 
                 const _Operation3& __z) 
    : _M_fn1(__x), _M_fn2(__y), _M_fn3(__z) { }
  typename _Operation1::result_type
  operator()(const typename _Operation2::argument_type& __x) const {
    return _M_fn1(_M_fn2(__x), _M_fn3(__x));  // 以参数分别调用仿函数 2、仿函数 3，并将结果作为仿函数 1 的两个参数，调用仿函数 1
  }
};

template <class _Operation1, class _Operation2, class _Operation3>
inline binary_compose<_Operation1, _Operation2, _Operation3> 
compose2(const _Operation1& __fn1, const _Operation2& __fn2, 
         const _Operation3& __fn3)
{
  return binary_compose<_Operation1,_Operation2,_Operation3>
    (__fn1 /* 仿函数 1 */, __fn2 /* 仿函数 2 */, __fn3 /* 仿函数 3 */);
}
```

## 8.4.4 用于函数指针

这类适配器使得开发者有能力将一般函数 (函数指针) 当成仿函数使用，使函数指针拥有适配能力。适配方法是将函数指针维护在类内，并重载类的 `operator()` 使其成为一个仿函数。

```c++
template <class _Arg, class _Result>  // 函数指针的参数类型 (1 个) 和返回类型
class pointer_to_unary_function : public unary_function<_Arg, _Result> {
protected:
  _Result (*_M_ptr)(_Arg);  // 一元函数的函数指针
public:
  pointer_to_unary_function() {}
  explicit pointer_to_unary_function(_Result (*__x)(_Arg)) : _M_ptr(__x) {}
  _Result operator()(_Arg __x) const { return _M_ptr(__x); }  // 以参数调用函数指针
};

template <class _Arg, class _Result>
inline pointer_to_unary_function<_Arg, _Result> ptr_fun(_Result (*__x)(_Arg))
{
  return pointer_to_unary_function<_Arg, _Result>(__x /* 一元函数指针 */);
}
```

```c++
template <class _Arg1, class _Arg2, class _Result>  // 函数指针的参数类型 (2 个) 和返回类型
class pointer_to_binary_function : 
  public binary_function<_Arg1,_Arg2,_Result> {
protected:
    _Result (*_M_ptr)(_Arg1, _Arg2);  // 二元函数的函数指针
public:
    pointer_to_binary_function() {}
    explicit pointer_to_binary_function(_Result (*__x)(_Arg1, _Arg2)) 
      : _M_ptr(__x) {}
    _Result operator()(_Arg1 __x, _Arg2 __y) const {
      return _M_ptr(__x, __y);  // 调用函数指针
    }
};

template <class _Arg1, class _Arg2, class _Result>
inline pointer_to_binary_function<_Arg1,_Arg2,_Result> 
ptr_fun(_Result (*__x)(_Arg1, _Arg2)) {
  return pointer_to_binary_function<_Arg1,_Arg2,_Result>(__x /* 二元函数指针 */);
}
```

## 8.4.5 用于成员函数指针

这种适配器使得开发者能够将类的成员函数作为仿函数来使用。如果以虚函数作为仿函数，那么还可以直接完成多态调用。将成员函数进行封装后，该适配器可以直接作为参数使用到 STL 算法中。

无参成员函数：

```c++
template <class _Ret, class _Tp>  // 返回类型，以及对象的数据类型
class mem_fun_t : public unary_function<_Tp*,_Ret> {
public:
  explicit mem_fun_t(_Ret (_Tp::*__pf)()) : _M_f(__pf) {}
  _Ret operator()(_Tp* __p) const { return (__p->*_M_f)(); }  // 重载 operator()，用参数中的对象指针直接调用成员函数
private:
  _Ret (_Tp::*_M_f)();  // 函数指针，指向成员函数
};

template <class _Ret, class _Tp>
class const_mem_fun_t : public unary_function<const _Tp*,_Ret> {
public:
  explicit const_mem_fun_t(_Ret (_Tp::*__pf)() const) : _M_f(__pf) {}
  _Ret operator()(const _Tp* __p) const { return (__p->*_M_f)(); }
private:
  _Ret (_Tp::*_M_f)() const;
};

template <class _Ret, class _Tp>
class mem_fun_ref_t : public unary_function<_Tp,_Ret> {
public:
  explicit mem_fun_ref_t(_Ret (_Tp::*__pf)()) : _M_f(__pf) {}
  _Ret operator()(_Tp& __r) const { return (__r.*_M_f)(); }
private:
  _Ret (_Tp::*_M_f)();
};

template <class _Ret, class _Tp>
class const_mem_fun_ref_t : public unary_function<_Tp,_Ret> {
public:
  explicit const_mem_fun_ref_t(_Ret (_Tp::*__pf)() const) : _M_f(__pf) {}
  _Ret operator()(const _Tp& __r) const { return (__r.*_M_f)(); }
private:
  _Ret (_Tp::*_M_f)() const;
};
```

带参成员函数：

```c++
template <class _Ret, class _Tp, class _Arg>  // 返回类型，对象类型，成员函数参数类型
class mem_fun1_t : public binary_function<_Tp*,_Arg,_Ret> {
public:
  explicit mem_fun1_t(_Ret (_Tp::*__pf)(_Arg)) : _M_f(__pf) {}
  _Ret operator()(_Tp* __p, _Arg __x) const { return (__p->*_M_f)(__x); }  // 重载 operator()，直接用对象指针调用成员函数，并传入参数
private:
  _Ret (_Tp::*_M_f)(_Arg);  // 函数指针，指向成员函数
};

template <class _Ret, class _Tp, class _Arg>
class const_mem_fun1_t : public binary_function<const _Tp*,_Arg,_Ret> {
public:
  explicit const_mem_fun1_t(_Ret (_Tp::*__pf)(_Arg) const) : _M_f(__pf) {}
  _Ret operator()(const _Tp* __p, _Arg __x) const
    { return (__p->*_M_f)(__x); }
private:
  _Ret (_Tp::*_M_f)(_Arg) const;
};

template <class _Ret, class _Tp, class _Arg>
class mem_fun1_ref_t : public binary_function<_Tp,_Arg,_Ret> {
public:
  explicit mem_fun1_ref_t(_Ret (_Tp::*__pf)(_Arg)) : _M_f(__pf) {}
  _Ret operator()(_Tp& __r, _Arg __x) const { return (__r.*_M_f)(__x); }
private:
  _Ret (_Tp::*_M_f)(_Arg);
};

template <class _Ret, class _Tp, class _Arg>
class const_mem_fun1_ref_t : public binary_function<_Tp,_Arg,_Ret> {
public:
  explicit const_mem_fun1_ref_t(_Ret (_Tp::*__pf)(_Arg) const) : _M_f(__pf) {}
  _Ret operator()(const _Tp& __r, _Arg __x) const { return (__r.*_M_f)(__x); }
private:
  _Ret (_Tp::*_M_f)(_Arg) const;
};
```

封装后的辅助函数：

```c++
// Mem_fun adaptor helper functions.  There are only two:
//  mem_fun and mem_fun_ref.  (mem_fun1 and mem_fun1_ref 
//  are provided for backward compatibility, but they are no longer
//  part of the C++ standard.)

template <class _Ret, class _Tp>  // 返回值类型、对象类型作为泛型参数
inline mem_fun_t<_Ret,_Tp> mem_fun(_Ret (_Tp::*__f)())  // 成员函数指针作为函数参数
  { return mem_fun_t<_Ret,_Tp>(__f); }

template <class _Ret, class _Tp>
inline const_mem_fun_t<_Ret,_Tp> mem_fun(_Ret (_Tp::*__f)() const)
  { return const_mem_fun_t<_Ret,_Tp>(__f); }

template <class _Ret, class _Tp>
inline mem_fun_ref_t<_Ret,_Tp> mem_fun_ref(_Ret (_Tp::*__f)()) 
  { return mem_fun_ref_t<_Ret,_Tp>(__f); }

template <class _Ret, class _Tp>
inline const_mem_fun_ref_t<_Ret,_Tp> mem_fun_ref(_Ret (_Tp::*__f)() const)
  { return const_mem_fun_ref_t<_Ret,_Tp>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline mem_fun1_t<_Ret,_Tp,_Arg> mem_fun(_Ret (_Tp::*__f)(_Arg))
  { return mem_fun1_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline const_mem_fun1_t<_Ret,_Tp,_Arg> mem_fun(_Ret (_Tp::*__f)(_Arg) const)
  { return const_mem_fun1_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline mem_fun1_ref_t<_Ret,_Tp,_Arg> mem_fun_ref(_Ret (_Tp::*__f)(_Arg))
  { return mem_fun1_ref_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline const_mem_fun1_ref_t<_Ret,_Tp,_Arg>
mem_fun_ref(_Ret (_Tp::*__f)(_Arg) const)
  { return const_mem_fun1_ref_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline mem_fun1_t<_Ret,_Tp,_Arg> mem_fun1(_Ret (_Tp::*__f)(_Arg))
  { return mem_fun1_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline const_mem_fun1_t<_Ret,_Tp,_Arg> mem_fun1(_Ret (_Tp::*__f)(_Arg) const)
  { return const_mem_fun1_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline mem_fun1_ref_t<_Ret,_Tp,_Arg> mem_fun1_ref(_Ret (_Tp::*__f)(_Arg))
  { return mem_fun1_ref_t<_Ret,_Tp,_Arg>(__f); }

template <class _Ret, class _Tp, class _Arg>
inline const_mem_fun1_ref_t<_Ret,_Tp,_Arg>
mem_fun1_ref(_Ret (_Tp::*__f)(_Arg) const)
  { return const_mem_fun1_ref_t<_Ret,_Tp,_Arg>(__f); }
```

---

