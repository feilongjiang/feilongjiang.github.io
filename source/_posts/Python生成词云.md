---
title: Python生成词云
date: 2017-03-25 15:02:21
category: Python
tags: 
- Python
- 词云
comments: true
---
![wordcloud](/images/imagesource/17-03-25-1.png )

## Python如何生成词云
最近看到词云这东西，感觉挺酷眩，就想自己实现以下:P
通过度娘发现可以用Python库来生成词云， So Let's go!
<!--more-->

### 用到的库
#### WorldCloud
- 官网: https://amueller.github.io/word_cloud
- github: https://github.com/amueller/word_cloud 


### 安装WordCloud

#### 通过PIP安装

    pip install wordcloud
#### 下载WHL包安装
当然可能通过PIP会安装出错，我们可以下[WHL包](http://www.lfd.uci.edu/~gohlke/pythonlibs/#wordcloud)手动安装

下载完成后，再使用PIP命令安装：

    pip install yourfilepath\wordcloud‑1.3.1‑cp36‑cp36m‑win_amd64.whl
    
其中， yourfilepath是你电脑中whl文件存放的位置，后面的包名为你所下载的whl包，我下的是`wordcloud‑1.3.1‑cp36‑cp36m‑win_amd64.whl`。

### 使用WordlCloud
安装完成后，就可以用了。可以先测试以下是否安装成功：

    from wordcould import WordCloud
如果没有报错，表示安装成功了。

#### 源代码
下面就是源码部分了:
``` python
# -*- coding: utf-8 -*-
from wordcloud import WordCloud
import matplotlib.pyplot as plt
from scipy.misc omport imread

# 读入一个文件
text = open('your text file.txt', 'r').read()
# 读入图片
bg_pic = imread('your picture.png')
# 配置词云参数
wc = WordCloud(
    # 设置背景色,我这里设置为了黑色
    background_color = 'black',
    # 设置词云形状，就是之前读入的图片
    mask = bg_pic,
    # 设置字体,字体路径要正确，不然会报错，最好和py文件放在一块
    font_path = 'Sketch Fine Serif.otf'
)
# 生成词云
wordcloud = wc.generate(text)
# 显示词云图片
plt.imshow(wordcloud)
plt.axis('off')
plt.show()
# 保存图片
wordcloud.to_file('wordcloud.jpg')
```

生成的词云效果如图:
![my wordcloud](/images/imagesource/17-03-25-2.jpg)
#### 参数

其中配置词云参数的时候，有多个参数可选：

Parameters
```
font_path : string
```
> Font path to the font that will be used (OTF or TTF). Defaults to DroidSansMono path on a Linux machine. If you are on another OS or don’t have this font, you need to adjust this path.

```
width : int (default=400)
```
> Width of the canvas.

```
height : int (default=200)
```
> Height of canvas

```
prefer_horizontal : float (default=0.90)
```
> The ratio of times to try horizontal fitting as opposed to vertical. If prefer_horizontal < 1, the algorithm will try rotating the word if it doesn’t fit. (There is currently no built-in way to get only vertical words.)

```
mask : nd-array or None (default=None)
```
> If not None, gives a binary mask on where to draw words. If mask is not None, width and height will be ignored and the shape of mask will be used instead. All white (#FF or #FFFFFF) entries will be considerd “masked out” while other entries will be free to draw on. [This changed in the most recent version!]

```
scale : float (default=1)
```
> Scaling between computation and drawing. For large word-cloud images, using scale instead of larger canvas size is significantly faster, but might lead to a coarser fit for the words.

```
min_font_size : int (default=4)
```
> Smallest font size to use. Will stop when there is no more room in this size.

```
font_step : int (default=1)
```
> Step size for the font. font_step > 1 might speed up computation but give a worse fit.

```
max_words : number (default=200)
```
> The maximum number of words.

```
stopwords : set of strings or None
```
> The words that will be eliminated. If None, the build-in STOPWORDS list will be used.

```
background_color : color value (default=”black”)
```
> Background color for the word cloud image.

```
max_font_size : int or None (default=None)
```
> Maximum font size for the largest word. If None, height of the image is used.

```
mode : string (default=”RGB”)
```
> Transparent background will be generated when mode is “RGBA” and background_color is None.

```
relative_scaling : float (default=.5)
```
> Importance of relative word frequencies for font-size. With relative_scaling=0, only word-ranks are considered. With relative_scaling=1, a word that is twice as frequent will have twice the size. If you want to consider the word frequencies and not only their rank, relative_scaling around .5 often looks good.

```
color_func : callable, default=None
```
> Callable with parameters word, font_size, position, orientation, font_path, random_state that returns a PIL color for each word. Overwrites “colormap”. See colormap for specifying a matplotlib colormap instead.

```
regexp : string or None (optional)
```
> Regular expression to split the input text into tokens in process_text. If None is specified, r"\w[\w']+" is used.

```
collocations : bool, default=True
```
> Whether to include collocations (bigrams) of two words.

```
colormap : string or matplotlib colormap, default=”viridis”
```
> Matplotlib colormap to randomly draw colors from for each word. Ignored if “color_func” is specified.

```
normalize_plurals : bool, default=True
```
> Whether to remove trailing ‘s’ from words. If True and a word appears with and without a trailing ‘s’, the one with trailing ‘s’ is removed and its counts are added to the version without trailing ‘s’ – unless the word ends with ‘ss’.

以上就是所有可以配置的参数，可以根据需要来用。

End~

---
[1]: 图片出处: https://github.com/amueller/word_cloud

