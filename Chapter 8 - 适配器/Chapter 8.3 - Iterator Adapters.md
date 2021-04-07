# Chapter 8.3 - Iterator Adapters

Created by : Mr Dk.

2021 / 04 / 07 22:48

Nanjing, Jiangsu, China

---

## 8.3.1 Insert Iterators

每个插入迭代器内部维护了一个容器。当用户对插入迭代器进行 **赋值** 操作时，就在插入迭代器内部转变为底层容器的插入操作。其它迭代器的功能都被关闭，前进 / 后退 / 取值 / 解引用等操作都是没有意义的。

后向插入迭代器。对迭代器的赋值操作将导致从容器的尾端插入元素。

```c++
template <class _Container>
class back_insert_iterator {
protected:
  _Container* container; // 指向底层容器
public:
  typedef _Container          container_type;
  typedef output_iterator_tag iterator_category; // 输出迭代器 (只写不读)
  typedef void                value_type;
  typedef void                difference_type;
  typedef void                pointer;
  typedef void                reference;

  explicit back_insert_iterator(_Container& __x) : container(&__x /* 指向底层容器 */) {}
  back_insert_iterator<_Container>&
  operator=(const typename _Container::value_type& __value) { // 重载赋值运算符
    container->push_back(__value); // 转而调用底层容器的 push_back() 函数
    return *this;
  }
  back_insert_iterator<_Container>& operator*() { return *this; } // 其它迭代器操作没用
  back_insert_iterator<_Container>& operator++() { return *this; }
  back_insert_iterator<_Container>& operator++(int) { return *this; }
};

template <class _Container>
inline back_insert_iterator<_Container> back_inserter(_Container& __x) {
  return back_insert_iterator<_Container>(__x); // 便于返回迭代器的函数
}
```

前向插入迭代器。对迭代器的赋值操作将导致从容器前端插入元素。

```c++
template <class _Container>
class front_insert_iterator {
protected:
  _Container* container;
public:
  typedef _Container          container_type;
  typedef output_iterator_tag iterator_category; // 输出迭代器 (只写)
  typedef void                value_type;
  typedef void                difference_type;
  typedef void                pointer;
  typedef void                reference;

  explicit front_insert_iterator(_Container& __x) : container(&__x /* 指向底层容器 */) {}
  front_insert_iterator<_Container>&
  operator=(const typename _Container::value_type& __value) { // 重载赋值运算符
    container->push_front(__value); // 转而调用底层容器的 push_front() 函数
    return *this;
  }
  front_insert_iterator<_Container>& operator*() { return *this; }
  front_insert_iterator<_Container>& operator++() { return *this; }
  front_insert_iterator<_Container>& operator++(int) { return *this; }
};

template <class _Container>
inline front_insert_iterator<_Container> front_inserter(_Container& __x) {
  return front_insert_iterator<_Container>(__x); // 便于返回迭代器的函数
}
```

跟随结点的插入函数。在迭代器指示的结点 (之前) 插入元素，插入成功后将返回的迭代器自增，指向原迭代器指示的元素。即插入位置永远随着构造对象时指定的那个迭代器。

```c++
template <class _Container>
class insert_iterator {
protected:
  _Container* container;
  typename _Container::iterator iter;
public:
  typedef _Container          container_type;
  typedef output_iterator_tag iterator_category; // 输出迭代器 (只写)
  typedef void                value_type;
  typedef void                difference_type;
  typedef void                pointer;
  typedef void                reference;

  insert_iterator(_Container& __x, typename _Container::iterator __i) 
    : container(&__x /* 指向底层容器 */), iter(__i /* 指示插入位置的迭代器 */) {}
  insert_iterator<_Container>&
  operator=(const typename _Container::value_type& __value) { 
    iter = container->insert(iter, __value); // 在迭代器指示的位置插入
    ++iter;                                  // 重新指向构造函数指定的迭代器
    return *this;
  }
  insert_iterator<_Container>& operator*() { return *this; }
  insert_iterator<_Container>& operator++() { return *this; }
  insert_iterator<_Container>& operator++(int) { return *this; }
};

template <class _Container, class _Iterator>
inline 
insert_iterator<_Container> inserter(_Container& __x, _Iterator __i)
{
  typedef typename _Container::iterator __iter;
  return insert_iterator<_Container>(__x, __iter(__i));
}
```

## 8.3.2 Reverse Iterators

反向迭代器将迭代器的移动行为翻转，改为从尾到头处理容器中的元素。反向迭代器内部维护了一个正向迭代器，只不过颠覆了正向迭代器的视角。

由于正向迭代器有 **左闭右开** 的特性，因此不能将反向迭代器简单实现为 `begin` 和 `end` 的互换。核心：先对类内的正向迭代器进行自减，然后再对该迭代器进行使用。这样 `rbegin()` 将会对应 `end()` 的前一个元素。说白了，左闭右开的视角不会变。

以下是 Random Access Iterators 对应的反向迭代器。由于支持随机存取，因此可以使用 `+=`、`-=` 等运算符。对于 Bidirectional Iterators 的反向迭代器，只对外提供重载后的 `++`、`--` 运算符。

```c++
class reverse_iterator {
  typedef reverse_iterator<_RandomAccessIterator, _Tp, _Reference, _Distance>
        _Self;
protected:
  _RandomAccessIterator current; // 维护正向迭代器，只需要颠覆其视角即可
public:
  typedef random_access_iterator_tag iterator_category; // 随机存取迭代器
  typedef _Tp                        value_type;
  typedef _Distance                  difference_type;
  typedef _Tp*                       pointer;
  typedef _Reference                 reference;

  reverse_iterator() {}
  explicit reverse_iterator(_RandomAccessIterator __x) : current(__x) {} // 可用一个已有的迭代器构造
  _RandomAccessIterator base() const { return current; }
  _Reference operator*() const { return *(current - 1); } // 访问当前迭代器所指的前一个元素
  pointer operator->() const { return &(operator*()); }   // 借用 * 运算符
  _Self& operator++() {
    --current; // 反向迭代器的 ++ 对应正向迭代器的 --
    return *this;
  }
  _Self operator++(int) {
    _Self __tmp = *this;
    --current;
    return __tmp;
  }
  _Self& operator--() {
    ++current; // 反向迭代器的 -- 对应正向迭代器的 ++
    return *this;
  }
  _Self operator--(int) {
    _Self __tmp = *this;
    ++current;
    return __tmp;
  }
  _Self operator+(_Distance __n) const {
    return _Self(current - __n); // 反向迭代器的 + 对应正向迭代器的 -
  }
  _Self& operator+=(_Distance __n) {
    current -= __n; // 反向迭代器的 += 对应正向迭代器的 -=
    return *this;
  }
  _Self operator-(_Distance __n) const {
    return _Self(current + __n); // 反向迭代器的 - 对应正向迭代器的 +
  }
  _Self& operator-=(_Distance __n) {
    current += __n; // 反向迭代器的 -= 对应正向迭代器的 +=
    return *this;
  }
    
  // 这里的第一个 * 使用的是当前类重载的 operator*，实际上访问的是 *(current - 1)
  // 第二个 * 特指 *this 为当前对象 (迭代器)
  // 这里的 + 也是使用当前类重载的 operator+，实际上表示 (current - __n)
  _Reference operator[](_Distance __n) const { return *(*this + __n); }
  // 实际上访问的是 *(current - __n - 1)
};
```

## 8.3.3 Stream Iterators

迭代器可以被绑定到一个 stream 上，使其拥有输入或输出的能力。

对于输入流迭代器来说，实际上就是在类内维护一个输入流 (`istream`)。对迭代器的 `++` 操作将会转而调用 `istream` 的 `>>` 操作获取输入。输入流是一个 Input Iterator，因为它是只读的。

```c++
template <class _Tp, class _Dist = ptrdiff_t> class istream_iterator;

template <class _Tp, class _Dist>
class istream_iterator {
  friend bool __STD_QUALIFIER
  operator== __STL_NULL_TMPL_ARGS (const istream_iterator&,
                                   const istream_iterator&);
protected:
  istream* _M_stream;   // 指向绑定的输入流
  _Tp _M_value;         // 暂存对读入的对象
  bool _M_end_marker;   // 流是否已结束
  void _M_read() {
    _M_end_marker = (*_M_stream) ? true : false;
    if (_M_end_marker) *_M_stream >> _M_value;   // 如果流未结束，则读取输入
    _M_end_marker = (*_M_stream) ? true : false;
  }
public:
  typedef input_iterator_tag  iterator_category; // 输入迭代器 (只读)
  typedef _Tp                 value_type;
  typedef _Dist               difference_type;
  typedef const _Tp*          pointer;
  typedef const _Tp&          reference;

  istream_iterator() : _M_stream(&cin), _M_end_marker(false) {}    // 默认输入流已结束 (通常用作 last 迭代器)
  istream_iterator(istream& __s) : _M_stream(&__s) { _M_read(); }  // 绑定输入流，并读入第一个元素
  reference operator*() const { return _M_value; }                 // 返回已读入并暂存的元素
  pointer operator->() const { return &(operator*()); }
  istream_iterator<_Tp, _Dist>& operator++() { 
    _M_read();  // 调用一次读取函数
    return *this;
  }
  istream_iterator<_Tp, _Dist> operator++(int)  {
    istream_iterator<_Tp, _Dist> __tmp = *this;
    _M_read();  // 调用一次读取函数
    return __tmp;
  }
};
```

由于构造该迭代器时会调用一次 `read()` 造成阻塞，所以一定要在确定需要的时候才定义输入流迭代器。示例：

```c++
istream_iterator<int> first(cin);
istream_iterator<int> last;  // (cin, false)
copy(first, last, ..., ...); // while first != last
```

对于输出流迭代器来说，类内绑定了一个 `ostream`。用户对迭代器的 `operator=` 操作，将会转而调用 `ostream` 的 `operator<<` 输出操作。

```c++
template <class _Tp>
class ostream_iterator {
protected:
  ostream* _M_stream;     // 指向绑定的输出流
  const char* _M_string;  // 每输出一次后跟着输出的分隔字符串
public:
  typedef output_iterator_tag iterator_category;  // 输出迭代器 (只写)
  typedef void                value_type;
  typedef void                difference_type;
  typedef void                pointer;
  typedef void                reference;

  ostream_iterator(ostream& __s) : _M_stream(&__s), _M_string(0) {} // 默认没有分隔字符串
  ostream_iterator(ostream& __s, const char* __c)                   // 指定输出流和分隔字符串
    : _M_stream(&__s), _M_string(__c)  {}
  ostream_iterator<_Tp>& operator=(const _Tp& __value) {  // 重载赋值运算符
    *_M_stream << __value;                   // 输出赋值给迭代器的数值
    if (_M_string) *_M_stream << _M_string;  // 如果有分隔字符串的话，输出分隔字符串
    return *this;
  }
  ostream_iterator<_Tp>& operator*() { return *this; }     // 迭代器运算操作无效
  ostream_iterator<_Tp>& operator++() { return *this; } 
  ostream_iterator<_Tp>& operator++(int) { return *this; } 
};
```

一种可能的使用方式：

```c++
ostream_iterator<int> out(cout, " ");
copy(vec.begin(), vec.end(), out);
```

---

