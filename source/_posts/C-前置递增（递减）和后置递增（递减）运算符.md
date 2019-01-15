---
title: C++前置递增（递减）和后置递增（递减）运算符
date: 2017-04-07 18:31:16
category: CPP
tags: C++
comments: true

---

在做C++ primer 第五版课后习题4.31时遇到一个问题：
> 本节的程序使用了前置版本的递增运算符和递减运算符，解释为什么要用前置版本而不用后置版本。要想使用后置版本的递增递减运算符需要做哪些改动？使用后置版本重写本节程序。

<!--more-->

源程序代码如下：
```c++
vector<int>::size_type cnt = ivec.size();
// 将把从size到1的值赋给ivec的元素
for(vector<int>::size_type ix = 0;
                    ix != ivec.size(); ++ix, --cnt)
    ivec[ix] = cnt;
```
可以看到源程序用的是前置递增和递减运算符
我根据源代码写了一下可运行的版本：
```c++
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	vector<int> ivec(10, 0);        // 初始化ivec
	vector<int>::size_type cnt = ivec.size();
	for (vector<int>::size_type ix = 0; 
	                    ix != ivec.size(); ++ix, --cnt)
		ivec[ix] = cnt;
	for (auto x : ivec)
		cout << x << " ";
	cout << endl;

	return 0;
}
```
输出的结果为
> 10 9 8 7 6 5 4 3 2 1

然后根据题目的意思，使用了后置版本的递增和递减运算符：
```c++
#include<iostream>
#include<vector>

using namespace std;

int main()
{
	vector<int> ivec(10, 0);        // 初始化ivec
	vector<int>::size_type cnt = ivec.size();
	for (vector<int>::size_type ix = 0; 
	                    ix != ivec.size(); ix++, cnt--)
		ivec[ix] = cnt;
	for (auto x : ivec)
		cout << x << " ";
	cout << endl;

	return 0;
}
```
输出的结果为
> 10 9 8 7 6 5 4 3 2 1

结果一样啊喂！(＃°Д°)
哪里需要修改了(╯‵□′)╯︵┻━┻
各种百度没有没百度到，突然想起来可以找课后练习答案啊！
找到了结果，还以为我错了，没想到答案居然是...

> We use prefix and not postfix, just because of the **Advice: Use Postfix   Advice: Use Postfix Operators only When Necessary**.
So, it's just a good habits. And there are no changes if we have to be made to use the postfix versions. Rewrite: 
```c++
for(vector<int>::size_type ix = 0; ix != ivec.size(); ix++, cnt--)  
    ivec[ix] = cnt; 
```
> This is not an appropriate example to discuss the difference of prefix and postfix.

果然不是我的问题 ( •̀ ω •́ )

