---
title: C++ string输入问题
date: 2017-03-31 14:38:28
category: CPP
tags:
- C++
- String
- vector
comments: true
---

## 问题描述

> 使用cin读入一组字符串并存入一个vector对象

<!--more-->

## 代码实现
使用如下代码
```c++
#include<iostream>
#include<vector>
#include<string>

using namespace std;

/*用cin读入一组字符串并把它们存入一个vector对象*/
int main()
{
	vector<string> text;
	string word;
	while (getline(cin, word))
	{
		text.push_back(word);
	}
	cout << "[";
	for (auto i : text)
		if (i == text.back())
			cout << i;
		else
			cout << i << ",";
	cout << "]";

	return 0;
}
```

在运行测试时发现了一个问题：
当输入为

> aa ss dd qq ww ee^Z (注意，^Z是在字符串输入完成后直接在后面加入的)

按回车后发现输出卡出了，并不会输出所输入的字符串

当输入为
> aa ss dd qq ww ee
^Z(换行后的^Z)

就可以正确输出了

## 解决办法
百思不得其解，遂百度之。
看到了两种解释，感觉都有道理，就都贴上来好了
1.

> 因为标准C++ I/O函数是有缓冲的，你输入aa dd ssctrl+z之后，cin调用read读到aa dd ss到缓冲中，返回aa给用户，再次调用直接返回dd，再次调用直接返回ss，从而忽略掉了ctrl+z，因为第一次read把它带过了。此刻调用cin，因为缓冲区空，于是read阻塞等待输入，你输入ctrl+z，read返回0，于是cin>>调用立即返回，cin状态为eof，while退出。

2.

> windows认为，如果缓冲中还有其它内容，ctrl+z不表示输入结束，仅代表当前行输入结束，只在单独一个ctrl+z的时候才表示输入结束。

意思就是说如果使用ctrl+z来结束输入的话，一定要是单独使用，不能在输入后面直接加。
