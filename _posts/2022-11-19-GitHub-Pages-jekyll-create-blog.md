---
title: GitHub Pages + jekyll + chirpy 搭建博客
author: hukeyi
date: 2022-11-19 17:06:00 +0800
categories: [Blogging]
tags: []
math: true
mermaid: true
toc: true
---

如果还未在 github 建立 `xxx.github.io` 仓库，则先根据 [GitHub: Creating a GitHub Pages site](https://docs.github.com/cn/pages/getting-started-with-github-pages/creating-a-github-pages-site)，把仓库建好后，再按下面执行。

# 1. 环境准备

我的环境：MacOS, zsh, homebrew

## 1.1 检查环境

参考[Jekyll 官方指南：环境准备](https://jekyllrb.com/docs/installation/)。shell 中依次输入以下 5 条命令，检查所需依赖是否已安装：

- Ruby: `ruby -v`

- RubyGems: `gem -v`

- GCC: `gcc -v` & `g++ -v`

- Make: `make -v`

## 1.2 下载 Ruby

[官方指南（MacOS）](https://jekyllrb.com/docs/installation/macos/)建议重新下载单独的较新版本的 Ruby，而非直接使用 macOS 原生的 Ruby。

按照[官方指南](https://jekyllrb.com/docs/installation/macos/)操作，执行 `ruby-install ruby` 时可能会遇见两个 bug。

### Bug 1: `Running Homebrew as root is extremely dangerous and no longer supported`

```zsh
(base) ➜  ~ ruby-install ruby
>>> Installing ruby 3.1.2 into /Users/uei/.rubies/ruby-3.1.2 ...
>>> Installing dependencies for ruby 3.1.2 ...
Error: Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.
```

解决方法：[stackoverflow: Cannot install homebrew packages because permission is denied](https://stackoverflow.com/questions/66544894/cannot-install-homebrew-packages-because-permission-is-denied)

1. 执行：`cd /usr/local && sudo chown -R $(whoami) bin etc include lib sbin share var Frameworks`

### BUG 2: `No such file or directory @ rb_sysopen all.bottle.tar.gz`

```zsh
==> Installing dependencies for openssl@1.1: ca-certificates
==> Installing openssl@1.1 dependency: ca-certificates
==> Pouring ca-certificates-2022-03-18.all.bottle.tar.gz
Error: No such file or directory @ rb_sysopen - /Users/uei/Library/Caches/Homebrew/downloads/e0b1b67cb3c1993f57c7b16dca8801a867bf6e9ef38ddb36d1f4dad6e598f843--ca-certificates-2022-03-18.all.bottle.tar.gz
!!! Installing dependencies failed!
```

解决方法（[知乎专栏：国内镜像网站同步问题](https://zhuanlan.zhihu.com/p/491515480)）

1. 首先，临时去除镜像：`export HOMEBREW_BOTTLE_DOMAIN=''`

2. 然后，重新执行 `ruby-install ruby`

检查安装结果：`ruby -v`

```zsh
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [x86_64-darwin21]
```

## 1.3 下载 jekyll

```zsh
gem install jekyll
```

检查安装结果：`jekyll -v`

# 2. 开始搭建网站

本节将会介绍如何按照这篇[参考文章](https://simondosda.github.io/posts/2021-09-14-blog-github-pages-2-setup.html)搭建默认主题 minima 的博客。

如果不想要默认主题，那么就在[官方指南：选择主题](https://jekyllrb.com/docs/themes/#pick-up-a-theme)中给的几个网站中选择自己喜欢的主题，把代码全部下载并复制到项目文件夹（xxx.github.io）中，再按照各自主题的指南操作就行了。

> 以下命令均在本地仓库文件夹环境 `(base) ➜ hukeyi.github.io git:(master) ✗ ` 下执行。
{: .prompt-info }

## 2.1 下载 jekyll bundler

```zsh
gem install jekyll bundler
```

## 2.2 创建新的 jekyll 项目

```zsh
jekyll new .
```

> 项目文件夹（`hukeyi.github.io`）必须清空，否则会报错：“Conflict: /Users/uei/hky/code/my-blog/hukeyi.github.io exists and is not empty.”报这个错的话把 `hukeyi.github.io` 文件夹内的所有文件删掉后再执行 `jekyll new .` 就好。
{: .prompt-danger }

安装后，项目文件夹长这样：

<img src="/assets/img/22-11/blog00.png" style="zoom:50%;" />

## 2.3 修改文件 Gemfile

打开 Gemfile，长这样：

```
...
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
gem "jekyll", "~> 4.3.1"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.5"
# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
# gem "github-pages", group: :jekyll_plugins
...
```

按照下面的【...】注释修改，修改完保存：

```text
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
# 【注释掉此行】gem "jekyll", "~> 4.3.1"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.5"
# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
【去掉下一行的注释，并且修改版本号（此处为 227，具体填多少见下面给的 Dependency versions)】
gem "github-pages", "~> 227", group: :jekyll_plugins
```

点开[Dependency versions: GitHub pages](https://pages.github.com/versions/)，找到 `github-pages` 那行对应的版本号：

<img src="/assets/img/22-11/blog01.png" style="zoom:50%;" />

修改完 Gemfile 文件长这样：

<img src="/assets/img/22-11/blog02.png" style="zoom:50%;" />

## 2.4 更新依赖

```zsh
bundle update
```

## 2.5 本地运行测试

```zsh
bundle exec jekyll serve
```

可能会报错：

### Bug: `bundler: failed to load command: jekyll`

```console
(base) ➜  hukeyi.github.io git:(master) ✗ bundle exec jekyll serve --livereload
Configuration file: /Users/uei/hky/code/my-blog/hukeyi.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /Users/uei/hky/code/my-blog/hukeyi.github.io
       Destination: /Users/uei/hky/code/my-blog/hukeyi.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.549 seconds.
 Auto-regeneration: enabled for '/Users/uei/hky/code/my-blog/hukeyi.github.io'
bundler: failed to load command: jekyll (/Users/uei/.gem/ruby/3.1.2/bin/jekyll)
/Users/uei/.gem/ruby/3.1.2/gems/jekyll-3.9.2/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

解决方法（[参考](https://talk.jekyllrb.com/t/load-error-cannot-load-such-file-webrick/5417/6)）

1. 首先，执行：`bundle add webrick`

2. 然后，重新执行：`bundle exec jekyll serve`

成功在本地运行：

```console
(base) ➜  hukeyi.github.io git:(master) ✗ bundle exec jekyll serve --livereload
Configuration file: /Users/uei/hky/code/my-blog/hukeyi.github.io/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /Users/uei/hky/code/my-blog/hukeyi.github.io
       Destination: /Users/uei/hky/code/my-blog/hukeyi.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.217 seconds.
 Auto-regeneration: enabled for '/Users/uei/hky/code/my-blog/hukeyi.github.io'
LiveReload address: http://127.0.0.1:35729
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

打开浏览器，输入 `localhost:4000`

<img src="/assets/img/22-11/blog03.png" style="zoom:50%;" />

## 2.6 把以上更改全部 push 到 github 远程仓库

浏览器输入 `hukeyi.github.io`

<img src="/assets/img/22-11/blog04.png" style="zoom:40%;" />

# 3. 选择主题

[官方指南：选择主题](https://jekyllrb.com/docs/themes/#pick-up-a-theme)中提供了几个主题网站供选择。本博客使用 [chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)。

chirpy 的[官方入门指南](https://chirpy.cotes.page/posts/getting-started/)写得很详细，按照指南一步一步做就好。

本博客因为已经搭建好环境，所以采用 zip 下载代码包，清空 `hukeyi.github.io` 项目文件夹后把代码包解压后复制到项目文件夹，再从官方入门指南的[安装选择 2](https://chirpy.cotes.page/posts/getting-started/#option-2-forking-on-github)开始执行，就成功了。
