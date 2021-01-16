---
title: 使用 Travis CI 实现 Hexo 博客自动构建和部署
date: 2019-01-16 13:54:52
updated: 2019-01-17 22:38:14
category: Tech
tags: [Travis-CI, Hexo]
comments: true
header_image: /images/bing/BingWallPaper-2019-01-16.jpg
---

![Travis CI](/images/imagesource/19-01-16/TravisCI-Full-Color.png)

昨天实现了用 Travis CI 对托管在 GitHub 上的 Hexo 博客进行自动部署。这么做的原因有两个：

1. 每次博客更新都要自己手动 `hexo d -g`，次数多了感觉有点麻烦，因此希望能用过自动化的方式部署，这样就只要专注博客内容更新，而非部署等操作。
2. 考虑到以后可能会更新设备(或者是多设备更新博客)，到时候又要重新安装一遍环境。设备之间的切换估计也比较麻烦。

因此考虑实现博客源代码放在 GitHub 仓库里，并实现博客更新自动部署。

<!--more-->

## Hexo 博客源代码 GitHub 托管

Hexo 是将 deploy 后生成的静态文件放在 GitHub Pages 上的，因此博客的源文件保存在本地。我们可以将博客源文件存放在同意仓库的另一个分支，比如我为个人博客仓库添加了 `source` 分支，用于存放源文件。

![github branches](/images/imagesource/19-01-16/2019-01-16-1.png)

具体操作如下：

### 添加分支

在 GitHub 仓库里选择添加一个新的分支，并设定分支名，我这里命名为 source。这是 source 分支和 master 分支的内容是一样的。

### 将仓库克隆至本地

在本地使用 `git clone 你的仓库地址` 将仓库克隆至本地，然后使用 `git checkout source` 命令，切换到新创建的分支。

### 在新分支中添加博客源文件

删除该分支下除了 `.git` 文件以外的所有内容，并将本地 Hexo 博客的相关文件夹复制进来。最后 source 分支下的内容应该是这样的：

![source file](/images/imagesource/19-01-16/2019-01-16-2.png)

然后 commit 更改，并 push 到远程仓库，这样我们就把本地的博客源文件存放到远程仓库的 source 分支里了。为下一步使用 Travis CI 自动化部署做准备。

## 使用 Travis CI 自动部署

### Travis CI 简介

Travis CI 提供持续集成服务(Continuous Integration, CI)。它可以绑定 GitHub 上的项目，只要有代码更新，就会自动抓取，并提供运行环境，执行测试，完成构建，还能部署到服务器。这里我们利用这个特性来为我们的 Hexo 博客实现远程构建并部署到 GitHub Pages。

### 配置 Travis CI

#### 网页端配置

首先进入 [Travis CI 官网](https://travis-ci.org/)，这里我们使用的是免费版的，因为考虑到一般放在 GitHub 上的博客都是公开的，所以不需要付费版本。如果有私有仓库要使用这种方式，可以使用 [付费版的 Travis CI](https://travis-ci.com/)。然后直接通过 GitHub 账户登陆即可，登陆后可以看到我们的共有仓库，找到博客的仓库，我这里是 `feilongjiang.github.io`，把旁边的勾勾上，然后点击旁边的 `Settings` 进入设置页面。

![Travis enable](/images/imagesource/19-01-16/2019-01-16-3.png)

在设置页面中，General 中只勾选 `Build pushed branches`，表示当有新的代码 push 到 GitHub 仓库时，自动执行构建任务。其他设置保持默认即可。

![Travis settings](/images/imagesource/19-01-16/2019-01-16-4.png)

接下来为 Travis 添加对 GitHub 仓库的读写权限。进入 [Personal access tokens](https://github.com/settings/tokens) 页面，点击 `Generate new token`，选择 token 权限(这里直选 repo 即可)，设置别名并生成。然后将生成的 token 值复制。

![generate token](/images/imagesource/19-01-16/2019-01-16-5.png)

接着在原来 Travis 的设置界面添加 token。如图所示：

![Travis token](/images/imagesource/19-01-16/2019-01-16-6.png)

在 Name 中填入 token 的别名，Value 中填入刚刚得到的 token，然后点击 Add 进行添加即可。注意 token 一旦生成，只能在生成时得到其值，后面无法查看。所以如果还有需要，可以记下来或者重新生成新的 token。

#### Travis 配置文件

接下来还需要编写 Travis 的配置文件，用于指定构建时使用哪些命令。配置文件名为 `.travis.yml`，是自动化构建的配置文件。文件内容示例如下：

```yml
language: node_js
sudo: required
node_js:
  - 7.9.0

# 指定缓存模块，可加快编译速度
cache:
  directories:
    - node_modules

# 指定博客源码分支，这里填入博客源码的分支名
branches:
  only:
    - source

before_install:
  - export TZ='Asia/Shanghai' # 更改时区
  - npm install -g hexo-cli

# S: Build Lifecycle
install:
  - npm install
  - npm install hexo-deployer-git --save

before_script:
 # - npm install -g gulp

script:
  - hexo clean
  - hexo generate

# 设置 git push 别名，邮箱。替换真实 token 到 _config.yml 文件中，然后 deploy 部署
after_script:
  - git config user.name "FreedomLy"
  - git config user.email "Freedom.JFL@gmail.com"
  - git clone https://github.com/feilongjiang/feilongjiang.github.io.git .deploy_git # 解决 commit 清空问题
  - cd .deploy_git
  - git checkout master
  - cd ../
  # 替换同目录下的_config.yml文件中gh_token字符串为travis后台刚才配置的变量，注意此处sed命令用了双引号。单引号无效！
  - sed -i "s/gh_token/${GH_TOKEN}/g" ./_config.yml # 这里的 GH_TOKEN 名字要和网页中定义的别名一致
  - cat ./_config.yml
  - hexo deploy
# E: Build LifeCycle
```

同时修改 Hexo 的 `_confi.yml` 文件中的 deploy 模块，将原来的

```yml
deploy:
  type: git
  repo: git@github.com:feilongjiang/feilongjiang.github.io.git
  branch: master
```

修改为

```yml
deploy:
  type: git
  repo: https://gh_token@github.com/feilongjiang/feilongjiang.github.io.git
  branch: master
```

这里的具体仓库地址需要替换成自己博客的仓库。下面解释一下两个配置文件中的 branch。在 `.travis.yml` 中的 branches 中填入的是存放博客源码的分支，而 `_config.yml` 中的 branch 则填入的是存放博客部署后产生的静态文件的分支。

然后将本地的更新 push 到源码所在的分支，就可以在 Travis 中看到 build 的条目了。

![Travis token](/images/imagesource/19-01-16/2019-01-16-7.png)

自此，使用 Travis CI 进行自动化构建并部署就完成了。以后只需编写 `.md` 的博客文件，然后 push 到远程仓库即可，无需本地生成和部署。这些操作交由 Travis CI 代为操作。

## 参考资料

[1]: [Hexo遇上Travis-CI: 可能是最通俗易懂的自动发布博客图文教程](https://blog.csdn.net/Xiong_IT/article/details/78675874)
[2]: [使用Travis CI自动部署Hexo博客](https://www.itfanr.cc/2017/08/09/using-travis-ci-automatic-deploy-hexo-blogs/)
[3]: [使用Travis Ci使hexo自动生成并部署](https://blog.xingoxu.com/2016/12/use-travis-ci-your-blog/)
[4]: [解决 Travis CI 总是更新旧博客的问题](https://wafer.li/Hexo/%E8%A7%A3%E5%86%B3%20Travis%20CI%20%E6%80%BB%E6%98%AF%E6%9B%B4%E6%96%B0%E6%97%A7%E5%8D%9A%E5%AE%A2%E7%9A%84%E9%97%AE%E9%A2%98/)