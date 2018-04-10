---
date: 2018-04-09 08:33
title: '线性表'
layout: post
author: 雪风
categories: IT
subtitle: ''
cover: 'https://qxuefeng.github.io/assets/img/books/c++-algorith/05/7acb0a46f21fbe0978bdddb26c600c338644adb0.jpg'
tags:  C++ algorithm
---

> **线性表(linear list)**是这样的数据对象，其实例形式为:  $(e_1,\ e_2,.....,\ e_n)$, 其中$n$是有穷自然数. $e_i$是表中的元素, $n$是表的长度. 当$n = 0$时，表为空; 当$n > 0$时, $e_1$是第一个元素, $e_n$是最后一个元素, $e_{n-1}$是$e_n$的直接前驱.

# 1. 线性表ADT
线性表应具备执行下列操作: 
- 创建一个线性表
- 确定线性表是否为空
- 确定线性表的长度
- 查找第k个元素
- 查找指定的元素
- 删除第k个元素
- 在第k个元素之后插入一个新元素

<center><h3><font color='red'>线性表的ADT描述</font></h3></center>
```cpp
抽象数据类型 LinearList {
实例
    0或多个元素的有序集合
操作
    Create(): 创建一个空线性表
    Destroy (): 删除表
    IsEmpty(): 如果表为空则返回true，否则返回false
    Length (): 返回表的大小(即表中元素个数)
    Find (k,x): 寻找表中第k个元素，并把它保存到 x 中；如果不存在，则返回false
    Search (x): 返回元素x在表中的位置；如果x 不在表中，则返回0
    Delete (k, x): 删除表中第k 个元素，并把它保存到 x 中；函数返回修改后的线性表
    Insert (k, x): 在第k个元素之后插入x；函数返回修改后的线性表
}
```
# 2. c++ class描述
c++ 支持两种类-抽象类和具体类. 通常使用抽象类来描述数据抽象类型ADT更为合适, 因为ADT只描述类的功能和操作，并没了实现的方法和过程的描述，这和抽象类的作用保持一致. 
<center><h3><font color='red'>线性表c++抽象类</font></h3></center>
```cpp
template<class T>
class LinearList {
public:
    virtual ~LinearList() { }
    
    /* 返回true, 当且仅当线性表为空 */
    virtual bool empty() const = 0;
    
    /* 返回线性表的元素个数 */
    virtual size_t size() const = 0;
    
    /* 返回索引为index的元素 */
    virtual T& get(int index) const = 0;
    
    /* 返回元素element第一次出现的索引 */
    virtual int indexOf(const T& element) const = 0;
    
    /* 删除索引为index的元素 */
    virtual void erase(int index) = 0;
    
    /* 把element插入线性表中索引为index的位置上 */
    virtual void insert(int index, const T& element) = 0;
} ;
```
# 3. 实现细节
## 3.1. 存储
要实现一个线性表，首先要考虑的就是一个元素的存储问题. 只有存储后，才能进行插入，查询等操作.
对于线性表， 可以通过数组来存储线性表的元素.若使用一维数组elements来存放线性表的元素.数组的位置有elements[0],...,elements[arrayLength-1], arrayLength即为数组的长度. 第i个元素在第i位置上, 可以很容易得到元素与位置的映射公式:
$$
location(i) = i
$$
## 3.2. 删除和插入
在长度为10的数组element中, 存储了5个元素的线性表[5,2,4,8,1].
当要删除线性表元素$e_i$,就需要把他右边的所有元素都向左移动一个位置. 例如, 删除$e_1=2$，必须把$e_2,..e_4$都像左移动一个位置.如下:
![]({site.url}assets/img/books/c++-algorith/05/20180409154633.png)
但在位置$i$插入一个元素,那么把$e_i$即其右边的元素在向由移动一个位置才能进行插入.
例如,在在位置2插入一个数字7，如下图:
![]({site.url}assets/img/books/c++-algorith/05/20180409154718.png)
## 3.3. 变长一维数组
还有个问题, 就是如果存储数组的空间只有10个元素的空间大小，那么在添加第11元素时就会出现没有空间而带来程序异常. 解决方法通常是对存储数组的空间进行扩大. 那么又会带来新的问题, 扩大的空间如果过小, 比如每插入一个元素重新扩大以下，就会非常影响重新的执行效率; 而扩大非常大的空间又会浪费内存. 因此, 为了平衡通常会在原来空间大小的基础上扩大一倍, 这样两者处于相对平衡.
```cpp
template<class T>
void changeLength1D(T*& a, int oldLength, int newLength)
{
    // 长度小于0返回false
    if(newLength < 0 || oldLength < 0) {
        /* 可以自己定义一个异常, 这里使用的是标准异常 */
        throw std::exception();
    }

    // 分配内存新空间, 失败抛出std::bad_alloc
    T* temp = new T[newLength];
    int number = std::min(oldLength, newLength);
    // 把a的newLength个元素拷贝到temp
    std::copy(a, a + number, temp);
    delete[] a;
    a = temp;
}
```

# 4. 实现
定义一个C++抽象类linearList的派生类arrayList, 实现线性表的所有操作.
<center><h3><font color='red'>arrayList的声明</font></h3></center>
```cpp
template<typename T>
class arrayList : public linearList<T>
{
public:
    // 构造函数, 拷贝构造函数和析构函数
    explicit arrayList(int initialCapcaity = 10);
    arrayList(const arrayList<T>&);
    ~arrayList() { delete[] element;}

    // ADT 方法部分实现
    bool empty() const { return listSize == 0;}
    int size() const { return listSize; }
    T& get(int index) const;
    int indexOf(const T& ele) const;
    void erase(int index);
    void insert(int index, const T& ele);

    // 额外方法
    int capacity() const { return arrayLength; }
    template<class U> friend std::ostream& operator<<(std::ostream&, const arrayList<U>&);

private:
    // 检查索引是否有效, 无效抛出异常
    void checkIndex(int index) const {
        if(index < 0 || index >= listSize)
            throw std::exception();
    }

    T* element;         // 存储元素的一维数组
    int arrayLength;    // 一维数组长度
    int listSize;       // 元素个数
};
```
## 4.1. 构造函数实现
```cpp
template<class T>
arrayList<T>::arrayList(int initialCapcaity)
{
    if(initialCapcaity < 1) {
        /* 可以自己定义一个异常, 这里使用的是标准异常 */
        throw std::exception();
    }

    arrayLength = initialCapcaity;
    element = new T[arrayLength];
    listSize = 0;
}

// 使用了c++11 特性的委托构造
template<class T>
arrayList<T>::arrayList(const arrayList<T>& List) :
    arrayList(List.arrayLength)
{
    listSize = List.listSize;
    std::copy(List.element, List.element + listSize, element);
}
```
arrayList(int initialCapcaity)时间复杂度都是$O(1)$, arrayList(const arrayList<T>& List)是$O(n)$

## 4.2. 基本方法实现
```cpp
template<class T>
T& arrayList<T>::get(int index) const
{
    checkIndex(index);
    return element[index];
}

template<class T>
int arrayList<T>::indexOf(const T& ele) const
{
    for(int i = 0; i < listSize; ++ i)
        if(element[i] == ele) return i;

    return -1;
}
```
get时间复杂度都是$O(1)$, indexOf是$O(n)$
## 4.3. 删除和插入实现
```cpp
template<class T>
void arrayList<T>::erase(int index)
{
    checkIndex(index);

    // 先调用析构删除位置元素
    element[index].~T();
    // 删除位置右边所有元素向前移动一位
    std::copy(element + index + 1, element + listSize, element + index);
    // 删除最后一个元素
    element[--listSize].~T();
}
template<class T>
void arrayList<T>::insert(int index, const T& ele)
{
    // 非法索引
    if(index < 0 || index > listSize)
        throw std::exception();

    // 线性表已满, 扩大线性表, 原来大小的两倍
    if(listSize == arrayLength) {
        changeLength1D(element, arrayLength, arrayLength*2);
        arrayLength *= 2;
    }

    // 向后移动一位
    std::copy(element+index, element+listSize, element+index+1);
    element[index].~T();
    element[index] = ele;
    listSize ++;
}
```
删除和插入操作时间都为时间复杂度为$O(n)$

## 4.4. 重载<<输出流实现
```cpp
template<class T>
std::ostream& operator<<(std::ostream& out, const arrayList<T>& x)
{
    for(int i = 0; i < x.listSize; ++ i)
        out << x.element[i] << " ";
    out << std::endl;
    return out;
}
```

## 4.5. 占用空间再优化
当空间因为多次插入扩大到10000后, 又进行了n次删除只剩10个元素时，剩下很多的空间都被浪费没有被有效利用. 所以需要某种策略, 减少被浪费的空间. 在进行erase操作时, 当listList < arrayList/4时数组长度缩小到max{initCapacity, arrayList/4}, 修改后的erase函数
```cpp
template<class T>
void arrayList<T>::erase(int index)
{
    checkIndex(index);

    // 先调用析构删除位置元素
    element[index].~T();
    // 删除位置右边所有元素向前移动一位
    std::copy(element + index + 1, element + listSize, element + index);
    // 删除最后一个元素
    element[--listSize].~T();

    // 缩小空间, 提高内存利用率
    if(listSize < arrayLength/4)
        changeLength1D(element, arrayLength, max(10, arrayLength/2));
}
```
## 4.6. arrayList迭代器
为了使arrayList能使用更加强大的函数, 需要为其创建一个迭代器.  在arrayList中添加代码实现迭代器功能
```cpp
// 迭代器
    class iterator
    {
    public:
        /* trains 设计模式 */
        typedef T value_type;
        typedef ptrdiff_t difference_type;
        typedef T* pointer;
        typedef T& reference;

        iterator(T* position = 0) : position(position) {}

        // 解引用操作
        T& operator *() const { return *position; }
        T* operator ->() const { return &*position; }

        // iterator ++ 和 -- 操作
        iterator& operator ++() { ++ position; return *this; }
        iterator operator ++(int) {
            iterator old = *this;
            ++ position;
            return old;
        }

        iterator& operator --() { -- position; return *this; }
        iterator operator --(int) {
            iterator old = *this;
            -- position;
            return old;
        }

        // eq and ne
        bool operator ==(const iterator right) const { return position == right.position; }
        bool operator !=(const iterator right) const { return !(*this == right); }


    private:
        /* 指向元素的指针 */
        T* position;
    };

    iterator begin() { return iterator(element); }
    iterator end() { return iterator(element+listSize); }
```
## 4.7. 测试
测试代码如下:
```cpp
#include <iostream>
#include "arraylist.h"
#include <numeric>


using namespace std;

int main()
{
    arrayList<int> arr(10);
    cout << "arr is empty: " << arr.empty() << endl;
    cout << "arr is capacity: " << arr.capacity() << endl;
    arr.insert(0, 1);
    cout << "insert 1: " << arr;
    arr.insert(0, 3);
    cout << "insert 2: " << arr;
    cout << "iterator test, sums: " << accumulate(arr.begin(), arr.end(), 0) << endl;
    arr.erase(1);
    cout << "after erase: " << arr;
    return 0;
}
```
![测试图]({site.url}assets/img/books/c++-algorith/05/20180409154700.png)
