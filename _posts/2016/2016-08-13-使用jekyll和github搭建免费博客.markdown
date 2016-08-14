---
layout: "post"
title: "使用Jekyll和Github搭建免费博客"
date: "2016-08-13 11:26"
---

## 写在前面

简单记录下我使用的搭建博客工具：

* 申请个人空间使用Dreamweaver、Flash等工具进行个人网站制作
* 新浪博客、网易博客、Blogger等在线博客工具
* Wordpress博客系统
* Github Pages + Jekyll (正在使用)

本篇内容将简单介绍下使用 Github Pages、Jekyll来搭建免费博客，后面还会介绍如何使用自己的域名和开通 HTTPS 访问支持。

## 为什么要用Github Pages和Jekyll

git是最好用的源代码管理软件，Github做为最好的开源社区之一，推出了Github Pages的业务，可以建立用户、机构组织、项目的网站，这一切还都免费，并且可以用git来进行网站源码的管理，极为方便，省却了自己部署服务器等繁琐事宜耗费的时间；使用Jekyll可以方便的进行静态类博客网站创建，再加上社区贡献的主题，总能选到你喜欢的主题，选不到自己做一个吧，静态类博客网站的缺点是对于标签、分类、评论等功能的支持，使用Jekyll都有对应的方法可以处理，这个我们以后再表。

## 准备工作

使用Github Pages和Jekyll进行博客网站搭建需要具备以下技能：

* 使用git进行代码管理
* 对于Github的使用有一定了解
* 可以较为熟练的使用 Ruby gem、bundle等命令

接下是准备工作：

```bash

# 去github上创建账号，并创建一个以 ${your-username}.github.io 为名称的空仓库

# 安装 bundler Jekyll
$ gem install bundler jekyll

# 假设博客网站在本地目录地址为 ~/Documents/sites/
$ cd ~/Documents/sites/ && mkdir ${your-username}.github.io
$ git init ${your-username}.github.io
$ cd ${your-username}.github.io

```

创建 Gemfile 文件，内容如下

```ruby

 source 'https://rubygems.org'
 gem 'github-pages', group: :jekyll_plugins

```

**注意**：众所周知的原因，rubygems.org的访问速度在国内很头疼，可以采用ruby.taobao.org的ruby镜像进行加速，加速方式如下

```bash

$ bundle config mirror.https://rubygems.org https://ruby.taobao.org

```

接下来安装依赖关系，创建Jekyll项目

```bash

$ bundle install

# 创建Jekyll项目
$ bundle exec jekyll new . --force

```

Run your Jekyll and start blogging!

```bash

bundle exec jekyll serve

```

这样一个本地的博客网站就运行起来了，访问 `http://localhost:4000` 便可看到你的博客网站了。

## 开始写博客

### 如何发布？

在建立的Jekyll项目里进行git初始化，并提交至在github上创建的地址便可发布了，使用 http://${your-username}.github.io 进行浏览。

### 如何写文章？

Github Pages和Jekyll对Markdown的支持及其良好，所以需要一款Markdown编辑器，在此建议使用Atom这款编辑器，再安装Markdown Writer插件，便可方便的进行文章的编写。

## 使用自有域名和HTTPS

首先需要去Godaddy、万网等域名注册商那里注册想要的域名（当然，这个是不免费的哈），然后参考Github Pages文档里的 [Using a custom domain with GitHub Pages][f7de4b2b]，进行域名的配置哈。建议给博客开通HTTPS服务，可以参考 [Set Up SSL on Github Pages With Custom Domains for Free][5a010e95] 来使用 Cloudflare 的免费CDN和HTTPS服务哈。


## Reference

* [Customizing GitHub Pages][9c1213d9]
* [Set Up SSL on Github Pages With Custom Domains for Free][5a010e95]
* [accent theme for Jekyll][96aa526e]
* [Jekyll][39261f22]

  [f7de4b2b]: https://help.github.com/articles/using-a-custom-domain-with-github-pages/ "Using a custom domain with GitHub Pages"
  [9c1213d9]: https://help.github.com/categories/customizing-github-pages/ "Customizing GitHub Pages"
  [5a010e95]: https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/ "Set Up SSL on Github Pages With Custom Domains for Free"
  [96aa526e]: https://ankitsultana.me/accent/ "accent theme for Jekyll"
  [39261f22]: http://jekyllrb.com/ "Jekyll"
