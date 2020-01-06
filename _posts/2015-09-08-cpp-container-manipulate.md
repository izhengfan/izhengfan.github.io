---
layout: cnpost
title: "C/C++：基本容器操作"
date: 2015-09-14 08:00:00
categories: cn
tags: C/C++
---


__目录__

* content
{:toc}

个人笔记，不定期更新。

### 容器简介

C++ 中的容器可以视为同一类的对象的集合，且包含了对集合内部元素的某些操作方法。常用容器有顺序容器和关联容器，顺序容器如向量（vector），array，列表（list）等，关联容器如 map，pair 等。其中 array 有点类似于 C 语言中的原生数组在 C++ 下的模板实现，向量则是面向对象的 C++ 编程中最常用的顺序容器。

### 数组

C 语言中，数组是定长的。元素的操作通过下标完成。数组的拷贝比较严格，一般要用 memcpy 函数。注意不可以直接用等号赋值数组名的方式进行数组拷贝。

C++ 中的 array 在使用性质上和原生数组颇有共通之处，这里不详细展开。

### 向量

向量操作比数组更灵活，长度可变，并支持丰富的泛型算法，因而最为通用。

### 迭代器

迭代器可以视为指向向量的某个元素的一个指针。创建一个迭代器的常用方法如下：

```cpp
vector<int> vec(10); // initialized size is 10
vector<int>::iterator iter = vec.begin();
```

更简单的写法是使用 auto 关键字，因为 `.begin()` 其实就是一个迭代器：

```cpp
auto iter = vec.begin();
```

迭代器是进行遍历操作的高效工具。对迭代器加一，即指向下一个元素。如，写向量的每个元素赋值：

```cpp
/* make the vector an ascending int array from 0 to 9 */
vector<int> vec(10);
auto iter = vec.begin();
while(iter != vec.end())
{
    *iter = iter - vec.begin();
    iter++;
}
```

### 基于向量的泛型算法

向量好用，因为支持丰富的泛型算法，主要来自 STL （标准模板库）。比如，查找算法：

```cpp
/* find a certain element in a vector and return its corresponding iterator */
/* if not found, return vec.cend() */
vector<int> vec = {1,2,3};
auto = find(vec.begin(),vec.end(),2);
if(auto != vec.cend())
    cout << "The location of 2 in the vector is " << auto - vec.begin();
else
    cout << "Not found!";
```

其他算法，比如求和、计数、拷贝等等，不赘述。需要注意：泛型算法基于模板，因而要注意使用对象，不能将算法用于一个由不支持该算法的类对象构成的向量。下面常见误操作中的数组无效拷贝即是一例。

### 关联容器

有时候我们希望把两个类型的对象关联在一起，这时要使用使用关联容器，如 std::map 类型。map 类型使用 key 和 key value 来构成关联关系：

```cpp
std::map <key_type, data_type, [comparison_function]>
```

通过 key，可以直接向 map 容器中添加或获取 key 所关联的元素：

```cpp
// require c++11 compilation
map<int, string> myMap;
vector<string> values = {"one","two","three"};  
myMap[1] = vector[0];
myMap[2] = vector[1];
cout << myMap[1] << endl;//this should print 'one'
```

当然，map 也支持迭代器操作，迭代器指向 first 或 second 可以获得 key 或 key value：

```cpp
// print all key values
for(auto it=myMap.begin();it!=myMap.end();it++){
    cout << it->second << endl;
}
// find a key and erase it
auto it = myMap.find(1);
if(it!=myMap.end()){
    cout << "myMap contains " << myMap[it->first] << 
    ", erase it." << endl;
    myMap.erase(it);
} else{
    cout << "No such value in myMap!" << endl;
} 
```

### 常见误操作

其实是我个人犯过的错误记录。

#### 迭代器过时

向量的内存空间是动态分配的，当对向量进行过元素增加或删减操作后，计算机会给向量重新分配内存空间，向量内元素的地址随之改变，迭代器也就过时了，类似于野指针，故需要重新指定。

```cpp
vector<int> vec = {1,2,3};
auto iter = vec.begin();
cout << "first element in vec is " << *iter << endl;
vec.push_back(4);
cout << "first element in vec is " << *iter << endl; // wrong, iterator is out of date
iter = vec.begin();
cout << "first element in vec is " << *iter << endl; // correct
```

有些操作会返回一个新迭代器，如 erase 函数在删除当前迭代器所指元素之后，返回一个指向下个元素的迭代器，在遍历操作时可利用。

```cpp
/* erase all even numbers in a vector */
vector<int> vec = {1,2,3,4};
auto iter = vec.begin();
while(iter != vec.end())
{
    if((*iter)%2==0)
    {	
        iter = vec.erase(iter);
    }
    else
    {
        iter++;
    }
}
```

#### 数组无效拷贝

如前所述，不可以通过等号赋值数组名进行数组拷贝。这一点比较容易注意到。但还是会有一些隐藏的坑，稍不注意就会失足。我们知道向量的拷贝是允许通过等号赋值向量名的方式实现的，假如我们构建一个元素为数组的向量，然后作如下拷贝：

```cpp
vec<double[3]> vec, vec2;
vec2 = vec; // wrong
```

这种操作是错误的。C++ 中向量之所以比数组更灵活，是因为向量操作往往是基于模板的。通过等号赋值向量名进行拷贝，也基于模板，对向量内的每个元素进行等号赋值拷贝。

因而，如果向量元素类型支持等号赋值拷贝，比如基础的整型和浮点型，那么用等号拷贝向量显然是合法的。如果向量元素类型不支持等号赋值拷贝，典型如数组，则等号拷贝向量不合法。此例也有利于更深地理解向量的性质和操作原理。
