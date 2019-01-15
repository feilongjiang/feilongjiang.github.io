---
title: C++ Primer 第五版 练习12.3
date: 2017-05-26 14:27:11
category: CPP
tags: C++ Primer
comments: true
---

## 问题

StrBlob需要const版本的`push_back`和`pop_back`吗？如果需要，添加进去。否则，解释为什么不需要。

<!--more-->

## 定义StrBlob类

首先，定义**StrBlob**类。类的定义如下：

```C++

/* StrBlob.h */
/* 编写你自己的StrBlob类，包含const版本的front和back */
#pragma once
#include<vector>
#include<string>
#include<initializer_list>
#include<memory>
#include<exception>

using std::vector;
using std::string;

class StrBlob {
public:
    using size_type = vector<string>::size_type;
    StrBlob() : data(std::make_shared<vector<string>>()) {}
    StrBlob(std::initializer_list<string> il)
        : data(std::make_shared<vector<string>>(il)) {}
    size_type size() cosnt { return data->size(); }
    bool empty() const { return data->empty(); }
    // 添加和删除元素
    void push_back(const string &t) { data->push_back(t); }
    void pop_back()
    {
       check(0, "pop_back on empty StrBlob");
       data->pop_back();
    }
    // 元素访问
    string &front()
    {
        // 如果vector为空，check会抛出一个异常
       check(0, "front on empty StrBlob");
       return data->front();
    }
    const string &front() const
    {
        check(0, "front on empty StrBlob");
       return data->front();
    }
    string &back()
    {
        check(0, "back on empty StrBlob");
        return data->back();
    }
    const string &back() const
    {
        check(0, "back on empty StrBlob");
        return data->back();
    }
private:
    void check(size_type i, const string &msg) const
    {
    if (i >= data->size())
        throw std::out_of_range(msg);
    }
private:
    std::shared_ptr<vector<string>> data;

```

## 测试

### 非const StrBlob对象

根据**StrBlob**的定义，我们来测试一下：
首先声明一个**StrBlob**对象**b1**：

    StrBlob b1 = {"a", "an", "the"};        // 非const对象

然后分别对**b1**调用`push_back()`和`pop_back()`：

    b1.push_back("test");       // 向b1添加元素test

此时**b1**保存的内容是 **a**, **an**, **the**, **test**;
接着就是

     b1.pop_back();         // 删除刚刚添加的元素

此时**b1**保存的内容是 **a**, **an**, **the**;
可以看到，我们定义的非const的**StrBlob**对象进行`push_back()`和`pop_back()`操作是没有问题的。

### const StrBlob对象

上面的操作都很顺利，那么当我们声明一个const **StrBlob**对象会怎么样呢？
接下来，我们声明一个const类型的**StrBlob**对象：

    const StrBlob cb = {"a", "an", "the"};       // const 对象
 
同样的，我们对**cb**调用`push_back()`和`pop_back()`，看看会发生什么：

    cb.push_back("test");       // 向cb添加元素test
运行一下程序，就会报错：
> *Error C2662 “void StrBlob::push_back(const std::string &)”: 不能将“this”指针从“const StrBlob”转换为“StrBlob &”*


    cb.pop_back();           // 删除元素

在运行程序，同样也会报错：
> *Error C2662 “void StrBlob::pop_back(void)”: 不能将“this”指针从“const StrBlob”转换为“StrBlob &”*

原因是我们在定义**StrBlob**类时，`push_back()`和`pop_back()`并没有添const版本。
那么，自然而然地，我们将添加const版本的`push_back()`和`pop_back()`，在StrBlob.h中添加以下代码:

```C++

/* StrBlob.h */
...
void push_back(const string &t) { data->push_back(t); }
void pop_back()
{
    check(0, "pop_back on empty StrBlob");
    data->pop_back();
}
// 添加const类型的push_back()和pop_back()
void push_back(const string &t) const 
{
    data->push_back(t);
}
void pop_back() const
{
    check(0, "pop_back on empty StrBlob");
    data->pop_back();
}
...

```

再运行一下程序，通过了，并且输出和之前非const版本的一样。

## 讨论

那么问题来了，我们到底需不需要const类型的`push_back()和pop_back()呢？

如果没有const的版本，那么对const类型的对象进行操作时，程序就会报错，貌似加const版本的`push_back()`和`pop_back()`是必须的。但是要注意的是：

**我们应该站在类的使用者的角度来看，而不是类的设计者的角度。虽然在类的具体实现中，数据成员是一个指向`vector<string>`的智能指针；但由于类的封装，在使用者看来，数据成员是`vector<string>`，他们并不知道具体实现使用了智能指针。那么当类的使用者声明类的常量对象时，他们期待的结果是`vector<string>`的内容不会改变。所以我们在设计这个类的时候，要考虑到类的使用者的真实意图，对于像`push_back()`和`pop_back()`这样会改变智能指针所指向的`vector<string>`内容的成员函数，我们不应该声明和定义成const版本。这样在类的使用者使用类的常量对象时，就不能调用`push_back()`和`pop_back()`成员函数，不能改变智能指针所指向的vector<string>的内容了，这正好与类的使用者意图相符。**

通过以上的考虑，我认为**StrBlob**不需要const版本的`push_back`和`pop_back`。


## 参考资料

 [1]:  [豆瓣 - 大家来讨论一下 exercise 12.3](https://www.douban.com/group/topic/61573279/)
 [2]:  [Stack Overflow - Operating on dynamic memory, is it meaningful to overload a const memeber function?](https://stackoverflow.com/questions/20725190/operating-on-dynamic-memory-is-it-meaningful-to-overload-a-const-memeber-functi)




End~

---
