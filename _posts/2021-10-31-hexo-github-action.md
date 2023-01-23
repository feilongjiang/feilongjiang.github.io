---
title: 通过 GitHub Actions 自动部署 Hexo 博客
date: 2021-10-31 22:08:52
category: [CICD, GHA]
tags: [hexo, github action, ci]
---

![Github Actions](/images/21-10-31/21-10-31-1.png)

Github 推出了 [GitHub Actions](https://github.com/features/actions) CI/CD，之前基于 Travis CI 做的持续部署，这次切换尝试切换到 GitHub Actions。

在 Marketplace/Actions 中找到了已有的 [Hexo Action](https://github.com/marketplace/actions/hexo-action)，本文就是基于它完成的。

## 准备工作

### 仓库准备
要实现自动部署，需要把博客的源文件也存放在仓库中。由于有些 TOKEN 可能会存放在配置文件中，建议单独为博客源文件创建一个 private 仓库，这样可以防止 TOKEN 泄漏。同时还需要一个存放 hexo deploy 以后所生成的文件仓库  `yourname.github.io`，用于发布博客。

> 如果不在意博客的配置是否公开的，可以直接将源文件 deploy 所生成的文件放在同一个仓库的不同分支下，即 `yourname.github.io` 下。
假设我们有两个仓库：
- `blog`: 存放了博客源码，称之为博客源码仓库
- `yourname.github.io`: 存放了 Hexo deploy 的文件，称之为博客仓库

博客源码仓库下以下内容是必须的: `scaffolds`，`source`，`themes`，`_config.yml`，`package.json`，`packge-lock.json`。由于我的博客之前是用 Travis CI 做部署的，因此只需将源文件复制过来即可。

如果之前用过 Travis CI 来进行自动部署，可能修改了博客目录下的 `_config.yml` 文件，需要将 deploy 下的 repo 改成博客仓库的 git 地址 (ssh 地址)，例如我的是:
```yml
deploy:
  type: git
  repo: git@github.com:feilongjiang/feilongjiang.github.io.git
  branch: master
```

后续操作皆基于上述两个仓库完成。

### Deploy Keys 和 Secrets

自动部署需要额外的部署密钥，通过 `ssh-keygen` 生成即可：

```
$ ssh-keygen -t rsa -C "username@example.com"
```

> 这里的邮箱请使用 GitHub 绑定的邮箱。

如果没有自定义输出文件名，上述命令会生成两个文件：`id_rsa.pub` 和 `id_rsa`，分别是公钥和私钥。将生成的公钥和私钥分别添加到博客仓库和博客源码仓库。

在博客源码仓库下，`Settings->Secrets` 页面点击 New repository secret，将私钥复制进去，并命名为 `DEPLOY_KEY`，然后点击 Add secret 即可。

> DEPLOY_KEY 的命名不可替换，会在后续的 GitHub Actions 中使用到。

在博客仓库下，`Settings->Deploy keys` 页面点击 Add deploy key，将私钥对应的公钥复制进去，命名随意，如 `HEXP_DEPLOY_KEY`，**记得要勾选 Allow write access**。

## 配置 GitHub workflows

在 `.github/workflows` 目录下新建 `deploy.yml`，使用 Hexo Action 提供的配置即可：

{% raw %}
```yml
name: Deploy

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    name: A job to deploy blog.
    steps:
    - name: Checkout
      uses: actions/checkout@v1
      with:
        submodules: true # Checkout private submodules(themes or something else).
    
    # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
    - name: Cache node modules
      uses: actions/cache@v1
      id: cache
      with:
        path: node_modules
        key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
        restore-keys: |
          ${{ runner.os }}-node-
    - name: Install Dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci
    
    # Deploy hexo blog website.
    - name: Deploy
      id: deploy
      uses: sma11black/hexo-action@v1.0.4
      with:
        deploy_key: ${{ secrets.DEPLOY_KEY }}
        user_name: your github username  # (or delete this input setting to use bot account)
        user_email: your github useremail  # (or delete this input setting to use bot account)
        commit_msg: ${{ github.event.head_commit.message }}  # (or delete this input setting to use hexo default settings)
    # Use the output from the `deploy` step(use for test action)
    - name: Get the output
      run: |
        echo "${{ steps.deploy.outputs.notify }}"
```

{% endraw %}

> `uses: sma11black/hexo-action@v1.0.4` 可以配置成最新的，本博客更新时，最新为 v1.0.4。

## 提交新博客并自动部署

上述配置完成后，可以尝试将本地改动提交的远程博客源码仓库，测试 GitHub Actions 是否正常执行。成功的话会如下图所示：

![github actions success](/images/21-10-31/21-10-31-2.png)

成功后，即可访问博客 `yourname.github.io` 查看更新后的内容了。

## 参考资料

1: [GitHub Action - Hexo CI/CD 🌱](https://github.com/marketplace/actions/hexo-action)