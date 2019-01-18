---
title: VSCode 常用插件记录
date: 2019-01-08 21:50:16
updated: 2019-01-14 10:47:08
category: Development
tags: VSCode
comments: true
header_image: /images/bing/BingWallPaper-2019-01-08.jpg
---

![vscode](/images/imagesource/19-01-08.png)
诈尸更新~~🤣🤣🤣
这篇文章用于记录我常用的 VSCode 插件，用以换新设备或者重装系统后重新安装插件使用。
<!--more-->

VSCode 下载地址：[Visual Studio Code](https://code.visualstudio.com/)

## 语言相关

### C/C++

推荐微软官方推出的 [C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)插件，支持智能提示和debug，很方便。

如何在 VSCode 上配置 C/C++ 环境可以参考知乎[Visual Studio Code如何编写运行C、C++？](https://www.zhihu.com/question/30315894/answer/154979413) 这个问题下的高赞回答。

通过 [vscode-fileheader](https://marketplace.visualstudio.com/items?itemName=mikey.vscode-fileheader) 插件可以一键为新建的 C/C++ 文件添加头部注释，如作者信息、文档新建日期、最后修改日期等。

### Python

同样推荐微软官方的 [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 插件。本来是民间大神写的，后来被微软收编了。只要安装好 [Python](https://www.python.org/downloads/) 环境，然后再装这个插件，写起 Python 代码来美滋滋！

同时再推荐 [Visual Studio IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode) 插件，可以用于 Python 的代码智能提示插件。利用 AI 让代码提示更加智能，可以根据语境来推荐调用函数等。支持都多种语言，包括 TypeScript/JavaScript，Java，Python 等语言。

## 文章书写

### Markdown

VSCode 对 Markdown 也有很好的支持。推荐如下几个插件：

1. [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)，为 Markdown 提供一站式支持，加快文章编写，提高行文效率。
2. [Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced)，增强 Markdown 的预览，同时支持输出 PDF (需要安装额外的工具) 以及 HTML 等格式。

### LaTeX

VSCode 还能用于书写 LaTeX，对于一些英文论文，使用 LaTeX 可以省去在排版上的精力。这里推荐 [TeX Live](https://www.tug.org/texlive/) 作为 TeX 环境。有了 TeX 环境后，再安装 [LaTeX Workshop](https://marketplace.visualstudio.com/items?itemName=James-Yu.latex-workshop) 插件，即可愉快地写 LaTeX 文档了。LaTeX Workshop 支持 TeX 智能提示、Tex 文件编译以及预览 PDF 文件等。同时还能自定义编译工具链。是一款十分强大的插件。

同时推荐 [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)，可以用于单词拼写检查，这样就不怕单词经常拼错了。支持多种文件类型，除了 LaTex 以外，还支持众多编程语言。也可以单独为某种语言设置是否开启单词拼写检查。当有自定义的单词时，还能够添加到词典中，添加到词典中的单词不会提示错误，如果误添加，可以在 setting 中删除。

## 美化相关

VSCode 默认有亮色和暗色主题，但用久了会出现审美疲劳。这里推荐几款 VSCode 主题和图标。

### 主题

1. [One Dark Pro](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme)，一款仿照 Atom 的 One Dark 主题
2. [Material Theme](https://marketplace.visualstudio.com/items?itemName=Equinusocio.vsc-material-theme)，Google Material Design 风格的主题，有多种颜色可以选择

### 图标

这里只推荐一个，个人最喜欢的一套图标 [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)，也是 Material Design 风格的。个人搭配 One Dark Pro 主题使用，非常耐看。

### 字体

VSCode 同时支持自定义字体，个人的字体设置为 `'Consolas', '思源黑体 Regular', monospace`，Consolas 比较适合编程，思源黑体用于中文字体显示。看起来比较舒服。

## 其他

### HTML 文件预览

写 HTML 文档时需要经常预览其样式和内容的改变，推荐使用 [View In Browser](https://marketplace.visualstudio.com/items?itemName=qinjia.view-in-browser)，通过快捷键 `Ctrl+F1` 即可在默认的浏览器中预览 HTML 文档了。

### GitHub

[GitHub](https://marketplace.visualstudio.com/items?itemName=KnisterPeter.vscode-github) 插件支持 GitHub pull/push 等源代码相关操作。

### 代码运行

[Code Runner](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner) 插件支持海量编程语言的一键运行。只要编写完代码，点击 run code 即可运行(前提是安装已经安装好对应的语言环境)。可以说是真神器。

### 语言环境

新版 VSCode 默认界面为英文，如果需要其他语言，需要安装对应的语言包，如[简体中文](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans)，安装完成后重新加载即可使用中文界面。其他语言同理。

### 同步设置

[Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync) 插件可以同步设置，只需创建一个 gist token，即可将 vscode 的配置信息上传至 [GitHub Gist](https://gist.github.com/) 中，当使用其他机器上的 vscode 时，可以通过这个插件来导入自己的配置信息。这个插件不仅同步设置，还可以同步按键设置、语言运行配置文件、代码段、插件以及插件配置、工作区文件。


以上就是个人常用的 VSCode 插件。涵盖了多个方面。如果以后有更好用的插件，再继续更新。