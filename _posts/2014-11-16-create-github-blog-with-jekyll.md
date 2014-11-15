---
layout: post
title: "Create github blog with Jekyll"
description: ""
category: 
tags: []
---
{% include JB/setup %}


### 一、Windows 下安装 Jekyll
http://jekyll-windows.juthilo.com/
Note: gem install jekyll -v 2.4.0 最新版本的 jekyll 会导致运行 jekyll serve 出错

### 二、创建 github 个人博客
http://jekyllbootstrap.com/usage/jekyll-quick-start.html
完成Host on GitHub in 3 Minutes即可，这时你已经在本地计算机上拥有博客的副本。

可以 jekyll serve 并到浏览器中查看 http://localhost:4000 本地页面是否能够访问

### 三、选择主题
http://themes.jekyllbootstrap.com/preview/tom/


### 三、Pygments 实现语法高亮

首先，安装 easy_install 用于安装 Pygments.

https://pypi.python.org/pypi/setuptools#windows-powershell-3-or-later

以 Powershell 为例，以管理员身份运行 Powershell, 并粘贴　(Invoke-WebRequest https://bootstrap.pypa.io/ez_setup.py).Content | python -
一旦安装完成，会在你安装 Python 的目录下生成 easy_install, 将此路径配置到环境变量中。
接着，运行 Powershell, 查看 easy_install --version 是否已经安装。
使用 easy_install 安装 Pygments: easy_install Pygments.

生成语法高亮的样式文件：
pygmentize -S default -f html > pygments.css
会在当前目录下生成 pygments.css 样式文件,将此文件拷贝到主题中的css样式文件中，并在引用文件中进行引用，如：_includes\themes\bootstrap-3\default.html 中进行引用

最后，打开_config.yml文件，在　highlighter: pygments　下添加　markdown: redcarpet，用于使用```的方式进行代码高亮。

