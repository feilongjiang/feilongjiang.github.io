---
title: C++中的explicit关键字
date: 2017-06-15 16:19:33
category: C++
tags: [c++]
---

## 隐式的类类型转换

在介绍explicit关键字之前，先来了解一下什么是隐式的类类型转换，C++ Primer第五版中的描述如下：
> 如果构造函数只接受一个实参，则它实际上定义了转化为次类类型的隐式转换规则，有时我们把这种构造函数称作**转换构造函数(converting constructor)**

也就是说编译器允许将构造函数参数类型通过隐式的转换，转化为一个类类型，以便为参数获取正确的类型。

## 转换示例

下面就给出一个例子，一个类的构造函数可以被用于隐式转换：

```c++
class Foo {
public:
	// 单形参构造函数，可以用作隐式转换
	Foo(int foo) : m_foo(foo) {}

	int getFoo() { return m_foo; }

private:
	int m_foo;
};

// 定义一个函数，接受Foo类型的形参
void Bar(Foo foo)
{
	int i = foo.getFoo();
}

// 主函数
int main()
{
	Bar(42);

	return 0;
}
```
可以看到在`main`函数中，传入`Bar`的并不是一个`Foo`的对象，而是内置类型`int`，但类`Foo`的构造函数接受一个`int`，因此这个构造函数可以用来将参数转换成正确的类型。

## 使用explicit抑制构造函数定义的隐式转换

为了防止编译器使用单参数构造函数进行隐式转换，需要在构造函数前加一个**explicit**关键字：

```c++
explicit Foo(int foo) : m_foo(foo) {}
```

这样就可以有效防止隐式转换。之前在`main`函数中`Bar(42)`就会报错：
> void Bar(Foo): 无法将参数1从"int"转换为"Foo"

进行如下修改，对传入的`int`进行显式转换
```c++
Bar(Foo(42))
```
这样就不会报错了。使用**explicit**关键字的原因是为了防止预想之外的情况发生。
下面举个例子（感觉不太合适）：
> 假设有一个MyString(int size)类，它有一个构造函数，可以构造给定size的字符串。还有一个函数print(cons MyString&)，然后调用print(3)。我们期望它能输出"3"，但是它实际上输出的是一个长度为3的空string。

## 结论

关于**explicit**的结论：

 - 关键字**explicit**只对一个实参的构造函数有效
 - 需要多个实参的构造函数不能用于执行隐式转换，所以无需将这些构造函数指定为**explicit**的
 - 只能在类内声明构造函数时使用**explicit**关键字，在类外部定义式不应重复

当我们用**explicit**关键字声明构造函数时，它将只能以直接初始化的形式(例如`string s("Hello")`)使用。而且编译器将不会在自动转换过程中使用改构造函数。

如果我们不希望函数进行隐式转换，那么最好将单形参构造函数定义为**explicit**的。这样可以避免错误，并且当需要转换时，我们也可以显式地构造对象。

## 参考资料

1: [Stack OverFlow - What does the explicit keyword mean?](https://stackoverflow.com/questions/121162/what-does-the-explicit-keyword-mean/121163#121163)
