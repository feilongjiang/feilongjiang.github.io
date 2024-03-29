---
title: 《Vim 教程》指令记录
date: 2019-05-17 13:54:10
category: [Linux, Vim]
tags: [vim]
---

之前跟着 [Vim](https://www.vim.org/download.php) 自带的 《Vim 教程》过了一遍常用的快捷键和命令，下面作一下记录，方便后续查找和回忆。

在命令行输入 `vimtutor` 进入 Vim 自带的教程。

## 基本操作

### 移动光标

移动光标分别为 `h`、`j`、`k`、`l` 键，当然也可以通过键盘上的方向键进行移动。不过考虑方向键离主键盘较远，因此使用`h`、`j`、`k`、`l`会更加高效。如果不确定当前的指令，只需要按下 `ESC` 键即可回到正常模式，然后再次输入指令。

### Vim 的进入和退出

在正常模式下，输入 `:q!` 然后再按下回车键即可退出当前打开的文档，该指令会丢弃之前对文件所作的所有改动。
如果是命令提示符中通过 `vimtutor` 进入教程的，这时会返回至命令行的界面，这时只需重新输入 `vimtutor`，即可重新进入教程，只不过之前对文档的修改都未得到保存（不管保存与否，通过 `vimtutor` 进入教程每次都会打开一个副本）。

> 若未对文档进行修改，输入 `:q` 即可退出 Vim，`:q!` 用于对文档有修改，但是不保存的情况，表示强制退出。

### 文本编辑之删除

在正常模式下（如果不知道是否处于正常模式，可以按下 `ESC` 来确保处于正常模式），可以按下 `x` 键来删除光标所在位置的字符。

### 文编编辑之插入

在正常模式下，按下 `i` 键来插入文本。这时会进入插入模式，当完成插入后，可按下 `ESC` 键返回正常模式。

### 文本编辑之添加

在正常模式下，按下 `A` 键来添加文本。与插入文本不同的是，添加文本操作会将光标移至当前行的行末。同样地，添加完成后，按下 `ESC` 键返回正常模式。

### 编辑文件

之前讲到如何退出 Vim，下面介绍如何保存修改后的文件。

在命令行输入 `vim <filename>` 并回车，会进入名为 `<filename>` 的文档，通过之前的命令对文档进行修改，修改完成后，输入 `:wq` 并按下回车键即可保存改动并退出 Vim。

## 删除与撤销

### 删除类命令

确保处于正常模式，输入 `dw` 即可从光标处删除至一个单词的末尾。

> 当我们输入时，字母 `d` 会同时出现在屏幕的最后一行。Vim 在等待输入字母 `w`。如果看到的是除 `d` 以外的其他字符，表明输入了错误的指令，这时可以按下 `ESC` 键，并重新输入。

### 更多删除类命令

在正常模式下，输入 `d$` 从当前光标删除到行末。

### 关于命令和对象

许多改变文本的命令都由一个操作符和一个动作构成。

使用删除操作符 `d` 的删除命令的格式如下：

**d motion**

其中：

- `d` 表示删除操作符
- `motion` 表示操作符的操作对象

基本的动作列表如下所示：

|操作对象|含义|
|:-:|:---|
|w|从当前光标当前位置直到下一个单词起始处，不包括它的第一个字符|
|e|从当前光标当前位置直到单词末尾，包括最后一个字符|
|$|从当前光标当前位置直到当前行末|

因此输入 `de` 会从当前光标位置删除到单词末尾。

> 若不输入操作符，只按下代表相应动作的键，则光标的移动将会按照上述动作进行。

### 使用计数指定动作

在动作前输入数字会使它重复那么多次。例如：

- 输入 `2w` 使光标向前移动两个单词。
- 输入 `3e` 使光标向前移动到第三个单词的末尾。
- 输入 `0` 移动光标到行首。

通过输入不同的数值，可以执行相应的操作。

### 使用计数以删除更多

使用操作符时输入数字可以使它重复那么多次。上面已经提到过删除操作和动作的组合，我们还可以在组合中动作之前插入一个数字以删除更多：

**d number motion**

例如：

- 输入 `d2w` 可以删除两个单词

### 操作整行

输入 `dd` 可以删除当前一整行。

鉴于整行删除操作的高频度，Vim 提供了简化的删除操作，我们只需在同一行上输入两次 `d` 即可删除光标所在的行了。同时可以在 `dd` 操作前加入数字，以删除多行，例如：

- 输入 `2dd` 可以删除两行。

### 撤销类命令

正常模式下，输入 `u` 来撤销最后执行的命令，输入 `U` 来撤销对整行的修改。例如按下了 `x` 删除了当前行的一个字符，那么可以通过 `u` 来撤销删除操作。若当前行执行了多个操作，需要全部撤销，则可通过 `U` 来撤销对当前行的所有修改。如果想要重做被撤销的命令，可以按下 `CTRL-R`，也即撤销掉撤销命令，回到原来修改后的状态。

## 置入、更改和替换

### 置入类命令

正常模式下，输入 `p` 将最后一次删除的内容置入光标后。如：

- 输入 `dd` 删除某一行
- 将光标移至准备置入的位置的**上方**
- 在正常模式下输入 `p` 将删除的一行粘贴置入

### 替换类命令

正常模式下，输入 `r` 和一个字符替换光标所在位置的字符。

### 更改类命令

正常模式下，要改变本文直到一个单词的末尾，可以使用 `ce`。同时也可以输入 `cw` 修改单词。具体操作为：

- 将光标移动至单词中需要修改的开始位置
- 输入 `cw` 并输入正确的单词
- 按下 `ESC` 键退出修改

> 请注意，`ce` 和 `cw` 命令不仅仅是删除了一个单词，它同时会进入插入模式。

### 使用 c 更改更多

更改类操作符可以与删除中使用的同样的动作配合使用。

更改类操作符的工作方式跟删除类是一致的，其操作格式为：

**c [number] motion**

动作参数（motion）也是一样的，比如 `w` 代表单词，`$` 表示行末等。

## 定位、查找和替换

### 定位及文件状态

正常模式下，输入 `CTRL-G` 显示当前编辑文件中当前光标所在行位置以及文件状态信息。

输入 `G` 可以使当前光标直接跳转到文件的最后一行。

输入 `gg` 可以使当前光标直接跳转到文件的第一行。

输入行号，然后再输入 `G` 即可使当前光标直接跳转到指定行。

### 搜索类命令

正常模式下，输入 `/` 加上一个字符串可以用以在当前文档中查找该字符串。

要查找同上一次的字符串，只需按下 `n` 键，如果要反向进行查找，则输入 `N`。

要反向查找字符串，可以用 `?` 代替 `/`。

要返回到之前的位置，可以按下 `CTRL-O`，重复按可以回退更多步。输入 `CTRL-I` 则会跳转到较新的位置，与 `CTRL-O` 执行的操作相反。

> 如果查找已经到达文件末尾，查找会自动从文件头部继续查找，除非 'wrapscan' 选项被复位。

### 配对括号的查找

正常模式下，输入 `%` 可以查找配对的括号 `)`、`]`、`}`。

若当前光标所在位置的字符为 `)`、`]` 或者是 `}`，输入 `%` 即可使光标跳转至配对的括号处，再次输入 `%` 即可返回之前括号的位置。

### 替换命令

正常模式下，输入 `:s/old/new/g` 可以替换 old 为 new。

若要替换两行之间出现的每个字符串，可以通过以下命令：

- 输入 `:#,#s/old/new/g`， `#,#` 表示替换操作的若干行的首尾行号
- 输入 `:%s/old/new/g`，表示替换整个文件中的每个匹配串
- 输入 `:%s/old/new/gc`，则会查找整个文件中的匹配串，并且对每个匹配串提示是否进行替换

## 文件相关操作

### 在 Vim 内执行外部命令的方法

在正常模式下，输入 `:!` 然后紧接着输入一个外部命令即可执行该外部命令。

如输入 `:!ls` 并按下回车，会列出当前目录的内容。

### 关于保存文件的更多信息

正常模式下，输入 `:w FILENAME` 可以将对文件的改动保存到 FILENAME 中。

若要删除保存的文件，可以通过执行外部命令的方法，输入 `:!rm TEST` 进行删除。

### 一个具有选择性的保存命令

在正常模式下，若要保存文件的部分内容，可以输入 `v motion :w FILENAME`。具体操作如下：

- 将光标移动至要保存内容的开始位置
- 接着按下 `v` 键进入可视模式，并继续移动光标至要保存内容的结束位置，这些内容会高亮显示
- 然后按下 `:`，这时屏幕底部会出现 `:'<,'>`
- 接着输入 `w FILENAME`，然后按下回车键
- 所选中的内容将会被写入 FILENAME 中。

> 按 `v` 键使 Vim 进入可视模式。我们可以移动光标使选取区域变大或变小。接着可以使用一个操作符对选中的文本进行操作。例如：按 `d` 键删除所选中的文本内容。

### 提取和合并文件

可以通过 `:r FILENAME` 命令向当前文件中插入另外文件的内容。

> 除了合并文件外，还可以读取外部命令的输出。例如：`:r !ls` 可以读取 ls 命令的输出，并将其放置在光标打下面。

## 打开、附加、替换、复制和设置类命令

### 打开类命令

正常模式下，输入 `o` 将在光标打下方打开新的一行并进入插入模式。

若要在光标的**上方**打开新的一行，则需要输入 `O`。

### 附加类命令

正常模式下，输入 `a` 将在光标之后插入文本。

> `a`、`i`、`A` 都会进入插入模式，唯一的区别在于字符插入的位置。`i` 将字符插入于光标之前，`a` 将字符插入于光标之后，而 `A` 则将字符插入于光标所在行的末尾。

### 另一个置换命令的版本

正常模式下，输入 `R` 可以连续替换多个字符。具体操作如下：

- 将光标移动至要替换内容的起始位置
- 输入 `R`，然后逐一输入要替换的字符
- 按下 `ESC` 键退出替换模式回到正常模式。

> 替换模式与插入模式类似，不过每个输入的字符都会删除一个已有的字符。

### 复制粘贴文本

正常模式下，使用操作符 `y` 复制文本，使用 `p` 粘贴文本。例如：

- 移动光标至所要复制内容的起始位置
- 按下 `v` 键进入可视模式
- 移动光标至所要复制内容的结束位置
- 输入 `y` 复制高亮文本
- 移动光标至要粘贴的位置，输入 `p` 以复制文本

> 可以把 `y` 当作操作符来使用。例如：`yw` 表示复制一个单词。

### 设置类命令选项

设置可使查找或者替换可忽略大小写的选项。例如：

- 要查找单词 ignore 可以在正常模式下输入 `/ignore` 并回车
- 然后设置 ic 选项（Ignore Case，忽略大小写），输入：`:set ic`
- 现在可以通过输入 `n` 键查找单词 ignore，且现在能够查找 Ignore 和 IGNORE
- 然后设置 hlsearch 和 insearch 这两个选项，输入：`set hls is`
- 现在在此输入查找命令：`/ignore`，可以发现符合要求的字符串高亮显示了
- 若要禁用忽略大小写，则输入：`:set noic`

输入 `:set xxx` 可以设置 xxx 选项，一些常用的选项如下：

|缩写|全称|含义|
|:--:|:--:|:--|
|ic|ignorecase|查找时忽略字母大小写|
|is|insearch|查找短语时显示部分匹配|
|hls|hlsearch|高亮显示所有的匹配短语|

> 若要移除匹配项的高亮显示，可以输入：`:nohlsearch`。在选项前加上 `no` 可以关闭选项。
> 如果想要仅在一次查找时忽略字母大小写，可以使用 `\c:/ignore\c` 并回车

## 其他命令

### 在线帮助系统

Vim 拥有一个细致全面的在线帮助系统，可以通过下述命令启动该系统：

- 按下 <HELP> 键
- 按下 <F1> 键
- 输入 `:help` 并回车

输入 `CTRL-W` 可以在不同窗口间切换

在 `:help` 命令中添加关键字，可以找到改关键字的帮助，如：

- `:help w`
- `:help c_CTRL-D`
- `:help insert-index`
- `:helo user-manual`

### 创建启动脚本

通过创建启动脚本，可以启用 Vim 的特性。Vim 的功能特性要比 Vi 多得多，但其中大部分都没有缺省启用。为了使用更多的特性，可以创建一个 vimrc 文件。

- 开始编辑 vimrc 文件，具体命令取决于您所使用的操作系统：
  - `:edit ~/.vimrc`，这是 Unix 系统所使用的命令
  - `:edit $VIM/_vimrc`，这是 MS-Windows 系统所使用的命令
- 接着读取 vimrc 示例文件的内容：
  - `:r $VIMRUNTIME/vimrc_example.vim`
- 保存文件，命令为：
  - `:write`

下次启动 Vim 时，编辑器就会有了语法高亮的功能。我们可以把喜欢的各种设置添加到这个 vimrc 文件中。要了解更多信息请输入 `:help imrc-intro`。

### 补全功能

使用 `CTRL-D` 和 `TAB` 可以进行命令补全。

> 补全对于许多命令都有效。只需尝试按 CTRL-D 和 TAB。它对于 :help 命令非常有用。