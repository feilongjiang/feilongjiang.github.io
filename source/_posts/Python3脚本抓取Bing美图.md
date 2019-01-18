---
title: Python脚本抓取Bing美图
date: 2017-03-23 14:26:18
updated: 2019-01-07 21:58:40
category: Python
tags: Python
comments: true
---
![post_background](http://api.dujin.org/bing/1920.php)

# 使用Python脚本下载Bing美图

[Bing首页](https://cn.bing.com)每天都会展示不同的图片，如果每天都去首页下载就比较麻烦了，于是乎想到了用Python脚本来代替，自动保存到图片目录下，省时省力。
<!--more-->

## Bing美图API

Bing官方有两个API：
xml版： http://cn.bing.com/HPImageArchive.aspx?idx=0&n=1
json版： http://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1
有了API就可以愉快地抓图了:D

## Python抓图

至于为什么要用Python，前段时间学习了廖大的[Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000),算是跟着把整个教程做完了，就放了一段时间。现在突然想弄个小工具来下Bing美图，嗯，就是它了！

## 用到的库

 - [Requests](http://cn.python-requests.org/zh_CN/latest/index.html)

## 爬图 (更新于2018-04-04)

这里我用了json版的API，使用`requests`的`json()`函数直接返回了API中的json字段，返回的类型是一个字典。我们需要的下载链接在`images`所对应的值里面。得到`images`的值后，他是一个列表，且里面嵌套了一个字典，我们提取出列表里的字典，然后根据键值`url`来得到图片的地址。改地址只是部分地址，我们还需要在地址前面加上`https:://www.bing.com`，将两个地址合起来就是图片的真实地址了，通过真实地址，就可以下载到bing主页的背景图片了。

## 源代码

```python

# -*- coding: utf-8 -*-
import requests
import os
import time


def get_content(url):
    """
    得到API返回的JSON数据
    :param url: API链接
    :return: 返回的是一个字典
    """
    try:
        r = requests.get(url)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.json()
    except:
        print("ERROR")


def get_img(url):
    """
    得到图片的内容
    :param url: 图片的真实链接
    :return: 返回的是byte类型的图片内容
    """
    try:
        r = requests.get(url)
        return r.content
    except:
        return None


def download(file_name, url):
    """
    根据得到的链接在指定位置保存图片
    :param file_name: 要保存图片的位置
    :param url: 图片的链接
    :return: 
    """
    if os.path.exists(r'C:\Users\Freed\Pictures\bingwallpaper'):
        print('Find Dir...')
    else:
        print("File dir did not exist, make dir...")
        try:
            os.mkdir(r'C:\Users\Freed\Pictures\bingwallpaper')
            print('Make dir success')
        except:
            print('Failed in make dir')
    if os.path.exists(r'c:\users\freed\pictures\bingwallpaper\\' + file_name):
        print('Image downloaded already')
    else:
        img = get_img(url)
        with open(r'c:\users\freed\pictures\bingwallpaper\\' + file_name,
                  'wb') as f:
            f.write(img)
        print('Download success')


if __name__ == '__main__':
    url = "https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1"
    content = get_content(url)
    url_dict = content['images'][0]
    download_url = 'https://www.bing.com' + url_dict['url']

    file_name = str("BingWallPaper-" + time.strftime('%Y-%m-%d',
                                                     time.localtime(time.time())) + '.jpg')
    download(file_name, download_url)
```

用Python命令运行文件即可。
到此大功告成!  :D
