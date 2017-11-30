# 关于

本站是我们用于记录日常工作中遇到的常见问题的处理方法的博客，目的是为了方便查阅以及帮助新人尽快上手。访问网址是： [https://sjscbjsjc.github.io](https://sjscbjsjc.github.io)

本博客使用了 gaohaoyang 的 jekyll 主题：[https://github.com/Gaohaoyang/gaohaoyang.github.io](https://github.com/Gaohaoyang/gaohaoyang.github.io)

# 添加贴子的方法

本博客使用 Ruby 的 jekyll 来生成静态页面，使用的时候需要先安装 ruby ，然后通过 gem 安装 jekyll 。

使用 git 将整个仓库 clone 后，在 _posts 目录下用 markdown 编辑内容，需要给内容加上一个类似下面的头：

```
---
layout: post
title:  "对这个 jekyll 博客主题的改版和重构"
date:   2016-03-12 11:40:18 +0800
categories: jekyll
tags: jekyll 端口 markdown Foxit RubyGems HTML CSS
author: Haoyang Gao
mathjax: true
---
```

下面这两行代码为产生目录时使用
```
* content
{:toc}
```

写好之后执行 jekyll build ，然后 jekyll s，就可以从本地访问了。访问正常后可以用 git push 到远程仓库。