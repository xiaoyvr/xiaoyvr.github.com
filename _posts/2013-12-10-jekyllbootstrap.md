---
layout: post
title: "这个博客是怎么搭建起来的"
description: "JekyllBootstrap"
category: "practice"
tags: [JekyllBootstrap,github,jekyll]
---
{% include JB/setup %}

当想出以这个题目作为在这个博客网站上的第一篇博客的时候，我的思考陷入了死循环。

为了写这篇博客，我需要搭建一个博客网站，然而在搭建这个博客网站之前，我是不知道如何写这篇博客的。到底是这个博客网站造就了这篇博客还是这篇博客催生了这个博客网站？

在计算机世界里，也有类似的鸡和蛋的问题。
> In computer science, bootstrapping is the process of writing a compiler (or assembler) in the target programming language which it is intended to compile. Applying this technique leads to a self-hosting compiler.

最直观的解决这个问题的方式就是用已经存在的语言Y先实现关于X的编译器，这个编译器实现好了之后，再用这个语言X重新实现一个X的编译器，就在这个时刻，自举[(bootstrapping)](http://en.wikipedia.org/wiki/Bootstrapping_%28compilers%29)完成了……

同样的，这个博客网站的搭建，也是参考了JekyllBootstrap的文档，GithubPages的文档和很多人的帖子才搭建起来的，然而既然已经搭建起来了，就先让他自举吧，这样关于这个博客的知识就变得自包含(self-hosting)了。

<!--more-->

### 搭建步骤

1. 你需要有github账号，你需要有ruby和git，没有就[brew](http://brew.sh/)一个。

1. 打开Jekyll-Bootstrap的[Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)，按照上面的步骤一步步走下来。

1. 打开http://USERNAME.github.io，你应该可以看到自己的博客了。想要在本地跑，在代码的根目录下运行`rake preview`，浏览器中打开http://localhost:4000，然后不断调试就OK了。

1. 在_config.yam中有很多fancy的选项，你可以按照需要根据Jekyll-Bootstrap的文档进行配置，这里记录一些需要注意的地方。
  * 在配置markdown引擎的时候，为了更好的支持GFM，要选择`markdown: redcarpet`，默认的那个对中文列表支持的不好，指定代码块的方式也很繁琐。
  * 有些theme对语法高亮没有太好的支持，需要手工添加/stylesheets/pygment_trac.css到相应theme的默认模版里。
  * 加入`excerpt_separator: <!--more-->`就可以支持自动摘要，只要在博客的正文中加入这个标记，然后在你的主页上使用`post.excerpt`即可。

更高级的技巧，就需要去研究[liquid](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers)的语法了。

接下来，在根目录下运行`rake create post="my first awsome post"`，去/_posts目录下找到这个markdown文件，用你喜欢的editor打开开始写博客吧，本地写好之后，发布博客就是commit之后push到github上，这一切是如此的简单。
