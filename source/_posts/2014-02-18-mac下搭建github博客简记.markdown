---
layout: post
title: "mac下搭建github博客简记"
date: 2014-02-18 19:56:50 +0800
comments: true
categories: 杂记
---

mac下搭建github博客简记
===========================

本文是博客搭建流程极简版，主要目的是记录下来给自己备忘，具体搭建流程请参考**破船之家**[\[1\]][1]、[\[2\]][2]

1、安装Ruby

安装RVM
    curl -L https://get.rvm.io | bash -s stable --ruby
安装最新版本的Ruby
    rvm list known
    rvm install latest-version
    rvm use latest-version
    rvm rubygems latest

2、安装Octopress

先将octopress从github上clone到本地
    git clone git://github.com/imathis/octopress.git octopress
安装相关依赖
    cd octopress
    gem install bundler
    bundler install
安装Octopress主题
    rake install

3、配置Octopress

略。

4、部署博客到github

在github创建username.github.com的repo，然后在Octopress主目录初始化仓库配置
    rake setup_github_pages
创建新博客
    rake new_post["title"]
找到创建的博客文件添加具体内容，然后使用下面命令发布
    rake generate
    rake deploy
以上步骤均成功后访问username.github.com可以看到正常页面。

5、解析godaddy域名到github博客

在godaddy上删除原有A记录新加一条A记录`@ 204.232.175.78`，再添加CNAME记录`http username.github.com`。另外github仓库下source目录新建CNAME文件，内容为要解析的域名。

[1]: http://beyondvincent.com/blog/2013/08/03/108-creating-a-github-blog-using-octopress/
[2]: http://beyondvincent.com/blog/2013/07/27/107-hello-page-of-github/
