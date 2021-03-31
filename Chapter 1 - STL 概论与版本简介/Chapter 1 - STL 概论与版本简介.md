# Chapter 1 - STL 概论与版本简介

Created by : Mr Dk.

2021 / 03 / 29 14:07

Nanjing, Jiangsu, China

---

## 1.1 STL 概论

制造 **可重复运用的东西**，防止大量程序员被迫从事大量重复的工作。

STL 的价值：

* 极具实用价值的零部件 + 整合的组织
* 高层次、以泛型思维为基础的、系统化的、条理分明的 **软件组件分类学**

C++ 的 **类** 使得程序员可以自定义类型，C++ 的 **模板** 使得程序员可以将类型 *参数化*。这两个特性为 STL 的形成提供了温床。

## 1.2 STL 六大组件 功能与运用

1. 容器 (containers)：各种数据结构；从实现角度看，是一种 *类模板 (class template)*
2. 算法 (algorithms)：各种常用算法；从实现角度看，是一种 *函数模板 (function template)*
3. 迭代器 (iterators)：泛型指针，扮演容器与算法的粘合剂；从实现角度看，是 **将指针相关操作予以重载** 的类模板
4. 仿函数 (functors)：行为类似函数；从实现角度看，是 **重载了 `operator()`** 的类或类模板
5. 适配器 (adapters)：用于修饰容器 / 仿函数 / 迭代器接口，用于暴露相同底层结构的不同功能
6. 分配器 (allocators)：负责空间的配置与管理；从实现角度看，是实现了 **动态空间分配 / 管理 / 释放** 的类模板

应用 STL 时，应遵照 C++ 规范，养成良好习惯，使用 **无扩展名的头文件**。

## 1.4 HP 实现版本

*Hewlett-Packard (HP)* 版本是所有 STL 实现版本的始祖。每个 HP STL 头文件上都有声明，允许任何人免费使用、拷贝、修改、传播、贩卖，唯一的条件是在所有文件中加上这个声明。属于开源范畴。

```c++
/*
 *
 * Copyright (c) 1994
 * Hewlett-Packard Company
 *
 * Permission to use, copy, modify, distribute and sell this software
 * and its documentation for any purpose is hereby granted without fee,
 * provided that the above copyright notice appear in all copies and
 * that both that copyright notice and this permission notice appear
 * in supporting documentation.  Hewlett-Packard Company makes no
 * representations about the suitability of this software for any
 * purpose.  It is provided "as is" without express or implied warranty.
 */
```

## 1.5 P.J.Plauger 实现版本

由 *P.J.Plauger* 开发，继承自 HP 版本，被 *Visual C++* 采用。

## 1.6 Rouge Wave 实现版本

由 *Rouge Wave* 公司开发，继承自 HP 版本，被 *C++Builder* 采用。

## 1.8 SGI STL 实现版本

由 *Silicon Graphics Computer Systems, Inc.* 公司开发，继承自 HP 版本。每份文件内除了 HP 的声明，还有 SGI 公司的版权声明。

```c++
/*
 *
 * Copyright (c) 1994
 * Hewlett-Packard Company
 *
 * Permission to use, copy, modify, distribute and sell this software
 * and its documentation for any purpose is hereby granted without fee,
 * provided that the above copyright notice appear in all copies and
 * that both that copyright notice and this permission notice appear
 * in supporting documentation.  Hewlett-Packard Company makes no
 * representations about the suitability of this software for any
 * purpose.  It is provided "as is" without express or implied warranty.
 *
 *
 * Copyright (c) 1996
 * Silicon Graphics Computer Systems, Inc.
 *
 * Permission to use, copy, modify, distribute and sell this software
 * and its documentation for any purpose is hereby granted without fee,
 * provided that the above copyright notice appear in all copies and
 * that both that copyright notice and this permission notice appear
 * in supporting documentation.  Silicon Graphics makes no
 * representations about the suitability of this software for any
 * purpose.  It is provided "as is" without express or implied warranty.
 */
```

SGI 版本被 GCC 采用，不论是在符号命名还是编程风格上，可读性都非常高。另外，SGI 为了具有高度移植性，考虑了不同编译器的不同编译能力。

## 1.9 可能令你困惑的 C++ 语法

### 1.9.2 临时对象的产生与使用

即所谓 **匿名对象**，STL 最常将此技巧应用于仿函数与算法的搭配上，比如 `greater<int>()`。`greater<int>` 实际上是一个仿函数类。`greater<int>()` 就是该仿函数类的一个临时对象。

### 1.9.4 increament / decrement / dereference 操作符

迭代器必须实现 **前进** (重载 `operator++`) 和 **取值** (重载 `operator*`) 两个功能，前进功能还要细分为前缀式和后缀式。对于有些支持双向移动的迭代器，还要具备 **后退** (重载 `operator--`) 的功能。

### 1.9.5 前闭后开区间表示法 [)

任何一个 STL 算法都要获得以一对迭代器指示的区间，以表示操作范围。迭代器表示的是一个 **左闭右开区间**。`[first, last)` 指示从 `first` 到 `last-1` 的元素，`last` 表示 **最后一个元素的下一个位置**。这种偏移一格的表示方法带来了许多方便。

### 1.9.6 function call 操作符 (`operator()`)

需要有一种特殊的东西来表达 **一组操作**，在 C 语言中，只能使用函数指针来实现，缺点是无法持有自己的状态，且不能达成可适配性。在 STL 中，以 **仿函数** 来实现这种功能。仿函数特指 **对某个类进行 `operator()` 重载**。通过将仿函数实例化为一个临时对象，就能方便地描述一个操作。

---

