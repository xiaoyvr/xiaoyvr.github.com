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

1. 在`_config.yml`中有很多fancy的选项，你可以按照需要根据Jekyll-Bootstrap的文档进行配置，这里记录一些需要注意的地方。
  * 在配置markdown引擎的时候，为了更好的支持GFM，要选择`markdown: redcarpet`，默认的那个对中文列表支持的不好，指定代码块的方式也很繁琐。
  * 有些theme对语法高亮没有太好的支持，需要手工添加`/stylesheets/pygment_trac.css`到相应theme的默认模版里。
  * 加入`excerpt_separator: <!--more-->`就可以支持自动摘要，只要在博客的正文中加入这个标记，然后在你的主页上使用`post.excerpt`即可。
  
1. 加入[Travis](https://travis-ci.org)支持。

 虽然说只是在写博客而已，然而所有的代码都在本地，当你提交代码到github之后，远程的jekyll会运行一个构建过程，也就是说，你写的东西需要与jekyll集成。如果你又使用了jekyll中更fancy的功能，那么更需要持续集成服务了。在本地调试是一个选择，然而更好的方式是搭建一个CI服务器，这样能够更好地保证你提交上去的东西是正常工作的，如果出了问题，也有一个地方能够看到log。这里，Travis是不二的选择，添加travis的支持，你需要做这些事情：
 1. 通过github注册一个travis账号（OAuth）
 1. 找到博客所对应的repository，进入service hook的设置，找到travis
 1. 点开，输入自己的用户名和travis的token，点上active，保存
 1. 再次进入travis的service hook界面，点test hook
 1. 在`_config.yml`的`exclude`配置中加入`'vender'`这一项
 1. 在根目录下建立`.travis.yml`文件，内容是
 
     ```yaml
     language: ruby
     script: "bundle exec jekyll build"
     ```
   1. 在你自己的readme.md文件的无所谓什么位置上加入
   
      ```
      [![Build Status](https://travis-ci.org/USERNAME/USERNAME.github.io.png)](https://travis-ci.org/USERNAME/USERNAME.github.io)
      ```
 好了，commit和push之后，你应该就能看到travis的绿色小图标在你的repostory上了，就好像这个一样[![Build Status](https://travis-ci.org/xiaoyvr/xiaoyvr.github.io.png)](https://travis-ci.org/xiaoyvr/xiaoyvr.github.io)。如果有什么错误的话，点这个图标去travis的页面上能够看到相应的log。

更多更高级的技巧，就需要去研究[Liquid](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers)的语法了。

接下来，在根目录下运行`rake post title="my first awsome post"`，去/_posts目录下找到这个markdown文件，用你喜欢的editor打开开始写博客吧，本地写好之后，发布博客就是commit之后push到github上，这一切是如此的简单。
