---
layout: post
title: "这个博客是怎么搭建起来的"
description: "JekyllBootstrap"
category: "practice"
tags: [JekyllBootstrap,github,jekyll]
---
{% include JB/setup %}

当想出以这个题目作为在这个博客网站上的第一篇博客的时候，我的思考陷入了死循环。

为了写这篇博客，我需要搭建一个博客网站，然而在搭建这个博客网站之前，我是不知道如何写这片博客的。到底是这个博客网站造就了这篇博客还是这篇博客催生了这个博客网站？

在计算机的世界里，也有类似的鸡和蛋的问题。
> In computer science, bootstrapping is the process of writing a compiler (or assembler) in the target programming language which it is intended to compile. Applying this technique leads to a self-hosting compiler.

最直观的解决这个问题的方式就是用已经存在的语言Y先实现关于X的编译器，这个编译器实现好了之后，再用这个语言X重新实现一个X的编译器，就在这个时刻，自举(bootstrapping)完成了……

相应的，这个博客网站的搭建过程，也是参考了JekyllBootstrap的文档，GithubPages的文档和很多人的博客才搭建起来的，然而既然已经搭建起来了，就先让他自举吧，这样关于这个博客的知识就变得自包含(self-hosting)了。

<!--more-->

### 搭建步骤

你需要有github账号，以下用**USERNAME**作为你账号的代称。

1. 创建一个新的repository，名字是USERNAME.github.io
1. 安装Jekyll-Bootstrap

```bash
git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com
cd USERNAME.github.com
git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
git push origin master
```
上面的命令执行之后，打开http://USERNAME.github.io，你应该可以看到自己的博客了
