---
layout: cnpost
title: "C++: 不可拷贝（noncopyable）类 "
date: 2016-11-20 01:00:00
categories: cn
tags: C/C++
---

__目录__

* content
{:toc}

### 拷贝

拷贝是任何一门编程语言都必不可少的操作。在 C++ 里，拷贝有等号拷贝和构造拷贝之分：

```cpp
Foo foo, foo2;
Foo foo2 = foo;  // 等号拷贝
Foo foo3(foo);   // 构造拷贝
```

等号拷贝是显式的，总得有个等号 `=` 在那才行。构造拷贝是隐式的，除了上面示例代码里那种直接写出构造函数的情况，在使用以值传递作为参数和以值返回的函数时，都会发生构造拷贝：

```cpp
/// 传入 foo 时，发生了一次实参到形参的构造拷贝
Foo func(Foo foo) {
    Foo foo2;
    // do somthing
    return foo2;
}
/// 返回时，发生了一次 foo2 到返回值的的构造拷贝
```

类的等号拷贝和构造拷贝都是可以重载的。如果不重载，默认的拷贝模式是对每个类成员依次执行拷贝。

### 什么时候需要不可拷贝类

考虑一种情况，我们要实现一个含有动态数组成员的类，其中动态数组成员在构造函数中 new 出来，在析构函数中 delete 掉。比如说这样一个矩阵类：

```cpp
template<typename _T>
class Matrix {
public:
   int w;
   int h;
   _T* data;
   
   // 构造函数
   Matrix(int _w, int _h): w(_w), h(_h){
      data = new  _T[w*h];
   }

   // 析构函数
   ~Matrix() {
       delete [] data;
   }
}
```

我们在这个类中用两个 int 数 `w` 和 `h` 代表矩阵的列数和行数，用一个长度为 `w*h` 的动态数组 `data[]` 存储矩阵数据。`data` 在构造函数中分配内存，在析构函数中释放内存。这里我们没有重载拷贝函数，那么如果拷贝这个类，会发生什么呢？

```cpp
/// 测试1：等号拷贝
void copy(){
    Matrix<double> m1(3,3);
    Matrix<double> m2(3,3);
    m1 = m2; 
}

/// 测试2：传参和返回中的构造拷贝
template<typename _T>
Matrix<_T> copy(Matrix<_T> cpy) {
    Matrix<_T> ret(3,3); 
    return ret;
} 
```

上面的测试 1 中，我们先构造了 `m1` 和 `m2` 两个 `Matrix` 实例，这意味着他们各自开辟了一块动态内存来存储矩阵数据。然后我们使用 `=` 将 `m2` 拷贝给 `m1`，这时候 `m1` 的每个成员（`w`，`h`，`data`）都被各自使用 `=` 运算符拷贝为和 `m2` 相同的值。`m1.data` 是个指针，所以就和 `m2.data` 指向了同一块的内存。于是这里就会出现两个问题：其一， 发生拷贝前 `m1.data` 指向的动态内存区在拷贝后不再拥有指向它的有效指针，无法被释放，于是发生了内存泄露；其二，在 `copy()` 结束后，`m1` 和 `m2` 被销毁，各自调用析构函数，由于他们的 `data` 指向同一块内存，于是发生了双重释放。

测试 2 中也有类似问题。当调用 `copy(Matrix<_T> cpy)` 时，形参 `cpy` 拷贝自实参，而 `cpy` 会在函数结束时销毁，`cpy.data` 指向的内存被释放，所以实参的矩阵数据也被销毁了——这显然是我们不愿意看见的。同样的，在返回时，`ret` 随着函数结束而销毁，返回值因为拷贝自 `ret`，所以其矩阵数据也被销毁了。

因此，对于像 `Matrix` 这样的类，我们不希望这种拷贝发生。一个解决办法是重载拷贝函数，每次拷贝就开辟新的动态内存：

```cpp
Matrix<_T>& operator = (const Matrix<_T>& cpy) {
    w = cpy.w;
    h = cpy.h;
    delete []  data;
    data = new _T[w*h];
    memcpy(data, cpy.data, sizeof(_T)*w*h);
    return *this;
}

Matrix(const Matrix<_T>& cpy):w(cpy.w), h(cpy.h) {
    data = new _T[w*h];
    memcpy(data, cpy.data, sizeof(_T)*w*h);
}
```

这样做也有不好的地方。频繁开辟动态内存，当数据量很大时（比如图像处理），对程序性能是有影响的。在接口设计的角度考虑，应该把这种拷贝操作以较明显的形式提供给用户，比如禁用等号拷贝，以直接的函数代替 `=` 操作：

```cpp
void copyFrom(const Matrix<_T>& cpy) {
    w = cpy.w;
    h = cpy.h;
    delete []  data;
    data = new _T[w*h];
    memcpy(data, cpy.data, sizeof(_T)*w*h);
}
```

再禁用构造拷贝，只允许用户以引用传递的办法在自定义函数中使用 `Matrix` 类。

那么，如何禁止拷贝操作呢？

### 实现不可拷贝类

#### 使用 `boost::noncopyable`

Boost 作为 C++ 万金油工具箱，在 `<boost/noncopyable.hpp>` 下提供了不可拷贝类的实现，使用起来也非常简单，让自己的类继承自 `boost::noncopyable` 即可：

```cpp
class Matrix : boost::noncopyable
{
    // 类实现
}
```

#### 声明拷贝函数为私有

如果不想用第三方库，自己实现呢？不妨先看一下 Boost 是怎么做的：

```cpp
private:  // emphasize the following members are private
    noncopyable( const noncopyable& );
    noncopyable& operator=( const noncopyable& );
``` 

嗯，直接把拷贝函数声明为私有的不就等于禁用了么，so smart！于是：

```cpp
template<typename _T>
class Matrix 
{
private:
    Matrix(const Matrix<_T>&);
    Matrix<_T>& operator = (const Matrix<_T>&);
}
```

#### C++ 11 下使用 delete 关键字

C++ 11 中为不可拷贝类提供了更简单的实现方法，使用 delete 关键字即可：

```cpp
template<typename _T>
class Matrix 
{
public:
    Matrix(const Matrix<_T>&) = delete;
    Matrix<_T>& operator = (const Matrix<_T>&) = delete;
}
```

### 其他

关于类似 `Matrix` 矩阵类的实现，更高级的做法是像智能指针一样封装其内部数据，用内部计数器来确定动态分配的成员是否要释放掉，不过这是另外一个问题了。


