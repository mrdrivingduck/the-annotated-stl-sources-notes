# Chapter 8.1-8.2 - Container Adapters

Created by : Mr Dk.

2021 / 04 / 07 20:44

Nanjing, Jiangsu, China

---

## 8.1 适配器的概念与分类

适配器在 STL 组件的灵活组合运用上扮演转换器的概念。Adapter 实际上也是一种设计模式，定义为：将一个 class 的接口转换为另一个 class 的接口，使原本因接口不兼容而不能合作的 classes 可以一起运作：

- 改变仿函数接口的，称为 function adapter
  - bind (绑定)
  - negate (否定)
  - compose (组合)
  - 对一般函数 / 成员函数进行修饰
- 改变容器接口的，称为 container adapter
  - queue / stack
- 改变迭代器接口的，称为 iterator adapter
  - insert iterators
  - reverse iterators
  - IO stream iterators

## 8.2 Container Adapters

stack 和 queue 两个适配器的底层都是 deque。这两个适配器通过屏蔽或保留 deque 的一些接口，形成了不同的功能。

```c++
template <class _Tp,
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(deque<_Tp>) >
class stack;

template <class _Tp, class _Sequence>
class stack {
// ...
protected:
  _Sequence c;
// ...
};
```

```c++
template <class _Tp,
          class _Sequence __STL_DEPENDENT_DEFAULT_TMPL(deque<_Tp>) >
class queue;

template <class _Tp, class _Sequence>
class queue {
// ...
protected:
  _Sequence c;
// ...
};
```
