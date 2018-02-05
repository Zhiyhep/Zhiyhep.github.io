---
layout: post
title: "Hello jekyll"
date: 2018-2-5 10:24
categories: jekyll
tags: jekyll
excerpt: 记录 jekyll 的创建过程。
author: zhiy
mathjax: true
---

* content
{:toc}

之前都是把自己学到的知识点记在笔记本上，记得东西多了以后既笨重又不好查找。所以决定搭建自己的博客，把各种笔记扔到网站上既方便又美观。用 Github+jekyll 可以很快实现一个轻量级的个人静态网站。在本地和 github 帐号下各保留一个副本，不仅可以随时随地修改还能方便别人访问。




## 搭建过程

搜索 &quot;jekyll&quot; 或 &quot;github pages&quot; 等关键字就能够找到很多相关的博客。但是比较早的博客内容很可能不再适用了。另外，不同博客的侧重点也不同，一些细节可能没有提到。所以我记录一下自己创建 jekyll 的过程，把自己遇到的坑都填上。
我的系统是 Ubuntu 14.04。  
主要步骤有： 安装 Ruby,  安装 RubyGems,  安装 jekyll,  使用 jekyll 模板， 上传到 github 。

### 安装 Ruby
我的机器上默认安装的是 ruby1.9.1， 而安装 jekyll 要求 ruby 版本在 2.0 以上, 所以需要升级 ruby 。在 [http://ftp.ruby-lang.org/pub/ruby/](http://ftp.ruby-lang.org/pub/ruby) 下载安装包, 我选择的是 2.5.0 版本。解压后执行如下命令：
```bash
$: cd ruby-2.5.0
$: ./configure --prefix=/usr/local --disable-install-doc --with-opt-dir=/usr/local/lib
$: make
$: make install
```
之后可以查看 ruby 版本：
```bash
$: ruby -v
ruby 2.5.0p0 (2017-12-25 revision 61468) [x86_64-linux]
```
### 安装 RubyGems
下载安装包：[https://rubygems.org/pages/download](https://rubygems.org/pages/download) 。
解压后，运行如下命令：
```bash
$: cd rubygems-2.7.4
$: ruby setup.rb 
```
安装完成后查看 rubygems 版本：
```bash
$: gem -v
2.7.3
```
不过明明下载的是 2.7.4 版本，安装后却变成了 2.7.3, 这一点我没搞懂。  
rubygems 默认从 [rubygems.org](https://rubygems.org/) 下载软件，在国内可能会无法访问。所以还需要把 gem 源改为国内镜像网站。
```bash
$: gem sources -r https://rubygems.org/
$: gem sources -a http://gems.ruby-china.org/
$: gem sources
*** CURRENT SOURCES ***

https://gems.ruby-china.org/
```

### 安装 jekyll
一键安装：
```bash
$: sudo gem install jekyll
$: sudo gem install pygments.rb // 代码高亮包
$: jekyll -v
jekyll 3.7.2
```
如果安装过程中出现错误，一般是没有安装依赖包。只要用 gem install 安装一下就可以了。  
至此，jekyll 安装完毕。接下来学习怎样在本地写 jekyll 博客并上传到 github 上就可以了。

### 使用 jekyll 模板
如果想要自己订制并灵活使用 jekyll 框架可以看这里：  
[英文版 jekyll 教程](https://jekyllrb.com/docs/home/)  
[对应的中文版教程](https://www.jekyll.com.cn/docs/home/)  
更方便和容易的做法是直接使用别人写好的模板。在 [jekyllthemes.org](https://jekyllthemes.org) 上可以找到很多漂亮的模板。在知乎话题[有哪些简洁明快的jekyll模板](https://www.zhihu.com/question/20223939)中，也有很多国内的小伙伴提供了自己的博客模板。我自己就选择了[高浩阳的模板](https://gaohaoyang.github.io)。选好心仪的模板后就可以拷贝到本地。
```bash
git clone https://github.com/Gaohaoyang/gaohaoyang.github.io.git
```
jekyll 的目录结构如下：  

![](/images/hellojekyll/jekyll_struct.png)  

其中 &quot;_config.yml&quot; 是全局配置文件（[参考资料](https://www.jekyll.com/cn/docs/configuration)）。一般来说，只需要修改 title 和链接地址即可。如果模板本身有评论或统计功能的话，最好在这里用 # 注释掉或者换成自己的 id 以免给作者带来困扰。  
在 &quot;_post&post; 文件夹下是博客内容。文章一般为 Markdown 或 textile 文件，标题格式为： 年-月-日-文件名。在每篇博客文章顶部必须包含YAML头信息（上下用三个破折号与文章内容隔离），用来说明改文章的布局、 标签、 作者、 创作时间等信息。头信息下面就是文章内容了， 可以用 Markdown 或 textile 语法来写。博客文件的命名和书写规范可以参考模板中的文件。  
写好文章后， 我们希望能够在本地预览一下。首先退出到博客的根目录（就是 _config.yml 所在的目录）下， 执行命令： jekyll serve, 会在 http://localhost:4000/ 端口运行开发服务器。用浏览器访问 http://localhost:4000/ 即可在本地预览。  
 
### 上传 github
首先在自己的 github 帐号下建立名为 username.github.io 的仓库。
然后在本地博客根目录修改远程仓库链接，然后上传到 github
```bash
git remote set-url https://github.com/username/username.github.io
git push -u origin master
```
最多几分钟后就可以在 https://username.github.io 看到自己的博客了。  
至此， 基于 jekyll 的博客框架就搭建完啦。以后只要在 &quot;_post&quot; 目录下添加博客文章，然后在根目录下用 git push 命令上传到 github 就好了。
 
