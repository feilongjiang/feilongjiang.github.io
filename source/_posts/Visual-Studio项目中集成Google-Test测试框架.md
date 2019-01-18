---
title: Visual Studio项目中集成Google Test测试框架
date: 2017-11-17 15:22:38
updated: 2019-01-07 21:44:48
category: CPP
tags: [C++, IDE]
comments: true
---

本文主要分成两个部分，第一部分介绍VS2015下Google Test的基本配置，第二部分讲一下我配置时遇到的问题。

<!--more-->

## VS2015下配置Google Test

### 下载Google Test

[Google Test][1]是Google旗下的一个单元测试框架。下载地址在[Release][2]中，在release页面下，有它的历史版本，我们下载最新的即可。

解压下载的文件，可以看到两个文件夹，这里我们要用的是**googletest**文件夹下的内容，其他的可以不用管。

![googletest](/images/imagesource/17-11-17/17-11-17-1.png)

解压出来的文件夹存放位置没有要求。我们主要用到的是googletest文件夹下的**include**和**src**。其中**include**用于配置项目时添加到包含目录中；**src**是Google Test框架的源码。

![src&include](/images/imagesource/17-11-17/17-11-17-2.png)

### 配置Google Test(2018-04-04更新)

感谢@LazyWolf Lin的指出，Google Test本身自带了msvc的项目文件，我们只需打开gtest提供的项目，编译生成得到静态库文件，然后在项目中设置运行库即可，不需要自己新建Google Test项目来生成静态库。具体的操作可以参考这一篇[博客][6]。

下载好文件后，我们开始配置Google Test。我们新建一个简单的项目作为测试。

#### 新建项目

打开VS2105，**新建项目->Win32控制台应用程序**，我们将项目名称设置为SimpleTest。然后选择**空项目**，完成即可。

![newproject](/images/imagesource/17-11-17/17-11-17-3.png)
![newproject1](/images/imagesource/17-11-17/17-11-17-4.png)

我们为项目添加两个文件：一个头文件，一个运行的文件。代码如下：

```c++
// simple_math.h
#pragma once
#include  <cmath>

double square(double num)
{
	return pow(num, 2);
}


// simple_math.cpp
#include "simple_math.h"
#include <iostream>

using namespace std;

int main()
{
	auto ans = square(10);
	return 0;
}
```

我们在`simple_math.h`中定义了一个`square`函数，返回传入的参数的平方值。并在`simple_math.cpp`中调用了该函数。可以按**Ctrl+F5**来测试一下是否运行正确。准备工作完毕，接下来就是配置Google Test了。

#### 配置Google Test

接下来我们右键最上面的**解决方案**，选择**添加->新建项目->Win32项目**，我们把新的项目名叫作GoogleTest，在应用程序类型那里选择**静态库**，取消预编译头，然后点击完成。

![googletestproject](/images/imagesource/17-11-17/17-11-17-6.png)

现在解决方案下面有两个项目，一个是我们之前创建的项目**SimpleTest**，一个是刚刚创建的**GoogleTest**。

![project2](/images/imagesource/17-11-17/17-11-17-7.png)

下面我们给GoogleTest添加包含目录，就是之前下载的googletest文件夹下的目录。**右键GoogleTest->属性->配置属性->VC++目录->包含目录**，我们在包含目录中添加Google Test相关文件。我们**googletest文件夹**和googletest目录下的**include**文件夹都添加进去，然后点击确定。

![includefile](/images/imagesource/17-11-17/17-11-17-8.png)

接着再给GoogleTest项目添加源文件。**右键GoogleTest的源文件->添加->现有项**，把之前提到的**src**目录下的**gtest_main.cc**和**gtest-all.cc**添加进去。

![addfile](/images/imagesource/17-11-17/17-11-17-9.png)

现在GoogleTest项目下有两个源文件。

![sourcefiles](/images/imagesource/17-11-17/17-11-17-10.png)

#### 生成静态库

上述配置完成后，我们**右键GoogleTest->生成**，就可以生成GoogleTest的静态库了。

![build](/images/imagesource/17-11-17/17-11-17-11.png)

#### 添加单元测试

下面我们测试一下GoogleTest是否配置正确，新建一个单元测试项目**UnittestSimpleTest**，**右键解决方案->添加->新建项目->Win32控制台应用程序**，选择空项目，然后完成即可。

现在我们的解决方案中共有三个项目：**GoogleTest，SimpleTest，UnittestSimpleTest**。

![project3](/images/imagesource/17-11-17/17-11-17-12.png)

同样，我们需要将googletest文件夹下的路径添加到UniitestSimpleTest的包含目录中。此外，我们还要添加SimpleTest的路径。

![includepath](/images/imagesource/17-11-17/17-11-17-13.png)

添加完包含目录后，再为其添加引用：**右键UnittestSimpleTest->添加->引用**，将**GoogleTest**和**SimpleTest**全勾上

![references](/images/imagesource/17-11-17/17-11-17-14.png)

单击确定即可。可以看到之前两个项目都作为引用添加到SimpleTest中了。

![references1](/images/imagesource/17-11-17/17-11-17-15.png)

#### 新建测试

我们现在可以添加源文件，编写单元测试了。新建一个源文件**test.cpp**，编写代码如下：
```c++
// test.cpp
#include "simple_math.h"
#include "gtest/gtest.h"

TEST(testSquare, mySquareTest)
{
	EXPECT_EQ(100, square(10));
}
```
然后我们将UnittestSimpleTest作为启动项目（**右键项目->设为启动项目**），然后运行即可。运行结果如图所示。

![result](/images/imagesource/17-11-17/17-11-17-16.png)

## 将现有的项目作为引用添加到其他项目中

在配置Google Test时，我新建一个项目按照上面的步骤可以使用单元测试，但是在给现有项目同样的配置时，却在添加完GoogleTest和单元测试后报错，一大片的LNK 2019的错误。

![errorlnk2019](/images/imagesource/17-11-17/17-11-17-17.png)

百度了半天没找到解决结果。后来看到了一篇文章，教我们怎么添加自己写的程序库，看了文章后我找到了链接错误的原因了：之前的项目没有作为**静态库**给新的项目引用，解决方法如下：**右键项目->属性->配置属性->常规**，将目标文件扩展名改为**.lib**，然后在下面的项目默认值中的**配置类型**改为**静态库**。

![buildall](/images/imagesource/17-11-17/17-11-17-18.png)

然后重新生成解决方案，没有报错，生成成功！然后就可以进行单元测试了。测试结果如图所示：

![result1](/images/imagesource/17-11-17/17-11-17-19.png)

在这边我只用到了静态库的生成和引用，以上所有操作都是debug模式下进行。

## 参考链接

1. [C++ TUTORIAL - GOOGLE TEST (GTEST)][3]
2. [带你玩转Visual Studio——带你发布自己的工程库][4]
3. [带你玩转Visual Studio——带你多工程开发][5]
4. [GTEST基础学习][6]

  [1]: https://github.com/google/googletest "Google Test"
  [2]: https://github.com/google/googletest/releases "Google Test Release"
  [3]: http://www.bogotobogo.com/cplusplus/google_unit_test_gtest.php "GTEST"
  [4]: http://blog.csdn.net/luoweifu/article/details/48895765 "发布自己的程序库"
  [5]: http://blog.csdn.net/luoweifu/article/details/48915347 "多工程开发"
  [6]: https://blog.csdn.net/lywzgzl/article/details/52203558 "GTEST基础学习"
