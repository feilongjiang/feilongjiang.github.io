---
title: C++ Primer 第五版 练习 12.33
date: 2017-06-07 22:39:56
category: CPP
tags: C++ Primer
comments: true
---

C++ Primer 第五版 第12章的动态内存终于（zou）看（ma）完（guan）了（hua）！！！记录一下本章的最后一个大练习——**文本查询程序**。
<!--more-->

## 文本查询程序

> 实现一个简单的文本查询程序，作为标准库相关内容学习的总结。程序允许用户在一个给定文件中查询单词。查询结果是单词在文件中出现的次数及其所在行的列表。如果一个单词在一行中多次出现，此行只列出一次。行会按照升序输出——即，第7行会在第9行之前显示，以此类推。

## 文本查询程序设计

设计两个类：`TextQuery`和`QueryResult`
其中`TextQuery`用来生成每个单词对应的行号以及进行相关的查询操作；
`QuerResult`用来保存查询结果，通过其成员函数`print()`输出查询结果。

## 查询程序的实现

Query.h
```C++
#pragma once
/* 在QueryResult类中添加名为begin和end的成员，返回一个迭代器，
   指向一个给定查询返回的行号set中的位置。再添加一个名为get_file的成员，
   返回一个shared_ptr，指向QueryResult对象中的文件
*/

#include"12.22.h"
// 定义了StrBlob类的头文件
using std::shared_ptr;

#include<iostream>
#include<fstream>
#include<map>
#include<set>

// 为了定义函数query的返回类型，这个定义是必须的
class QueryResult;
class TextQuery {
public:
	TextQuery(std::ifstream&);
	QueryResult query(const string&) const;

private:
 	// 输入文件
	shared_ptr<StrBlob> file;						   
	// 每个单词到它所在行号的映射
	std::map<std::string, 
		shared_ptr<std::set<StrBlob::size_type>>> wordmap;
};

class QueryResult {
	using qr_iter = std::set<StrBlob::size_type>::iterator;
	friend std::ostream &print(std::ostream&, QueryResult&);

public:
	QueryResult(string s, shared_ptr<std::set<StrBlob::size_type>> l, shared_ptr<StrBlob> f) :
		word(s), lines(l), file(f) {}

	qr_iter begin() const { return lines->begin(); }
	qr_iter end() const { return lines->end(); }

	shared_ptr<StrBlob> get_file() const { return file; }

private:
	// 查询单词
	string word;
	// 出现的行号
	shared_ptr<std::set<StrBlob::size_type>> lines;
	 // 输入文件
	shared_ptr<StrBlob> file;
};

std::ostream &print(std::ostream &os, QueryResult &query_result);

```

Query.cpp
```C++
#include "Query.h"
#include<iterator>
#include<sstream>
#include<algorithm>

//读取输入文件并建立单词到行号的映射
TextQuery::TextQuery(std::ifstream &ifs) : file(new StrBlob)
{
	// 保存行号
	StrBlob::size_type line_no{ 0 };
	// 对文件中的每一行
	for (string line; std::getline(ifs, line); ++line_no)
	{
		// 保存此行文本
		file->push_back(line);
		 // 将文本分解为单词
		std::istringstream stream(line);
		// 对行中每个单词
		for (string text, word; stream >> text; word.clear())
		{
			// 去除单词中的标点符号
			std::remove_copy_if(text.begin(), text.end(),
				std::back_inserter(word), ispunct);
			// 如果单词不在wordmap中，以之为下标在wordmap中添加一项
			auto &lines = wordmap[word];
			// 第一次遇到这个单词时，lines的指针为空
			// 分配一个新的set
			if (!lines) lines.reset(new std::set<StrBlob::size_type>);
			// 将此行号插入set中
			lines->insert(line_no);
		}
	}
}

QueryResult TextQuery::query(const string &sought) const
{
	// 如果未找到sought，将返回一个指向此set的指针
	static shared_ptr<std::set<StrBlob::size_type>>
		nodata(new std::set< StrBlob::size_type>);
	// 使用find而不是下标运算符来查找单词，避免将单词添加到wordmap中
	auto found = wordmap.find(sought);
	if (found != wordmap.end())
		// 找到单词
		return QueryResult(sought, found->second, file);
	else
		// 未找到
		return QueryResult(sought, nodata, file);
}

std::ostream & print(std::ostream &os, QueryResult &query_result)
{
	// TODO: 在此处插入 return 语句
	// 如果找到了单词，打印出现次数和所有出现的位置
	os << query_result.word << " occurs " << query_result.lines->size() << " "
		<< (query_result.lines->size() > 1 ? "times." : "time.") << std::endl;

	// 打印单词出现的每一行
	for (auto it = query_result.begin(); it != query_result.end(); ++it)
	{
		ConstStrBlobPtr p(*query_result.file, *it);
		os << "\t(line " << *it + 1 << ") " << p.deref() << std::endl;
	}
	return os;
}
```

## 测试
Query_test.cpp
```C++
#include"12.33.h"

void runQueries(std::ifstream &ifs)
{
	TextQuery text_query(ifs);
	do
	{
		std::cout << "Enter word to look for or q to quit: ";
		string word;
		if (!(std::cin >> word) || word == "q") break;
		print(std::cout, text_query.query(word)) << std::endl;
	} while (true);
}

int main()
{
	std::ifstream ifs("data/story.txt");
	runQueries(ifs);

	return 0;
}
```

## 总结
这里用到了之前几节定义的`StrBlob`来代替`vector<string>`来保存每一行的内容。
算是一个比较综合的练习了。刚开始做没什么思路，多亏了[GayHub](https://github.com/pezy/CppPrimer)（雾）上的大神，才完成了这个练习。革命尚未成功，同志还需努力!
(ง •̀_•́)ง 



End~

---