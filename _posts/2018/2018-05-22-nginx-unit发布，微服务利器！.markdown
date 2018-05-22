---
layout: "post"
title: "Nginx Unit发布，微服务利器！"
date: "2018-05-22 10:45"
---

该文章首发于我的微信公众号：程序员精进 ，欢迎关注订阅。

> 摘要：NGINX, Inc.公司发布了全新动态Web和应用服务器 Nginx Unit 1.0，轻量动态且支持多语言及版本。本文将介绍下Nginx Unit是什么，分析下它的作用以及它会给市场带来怎样的变化。注：目前Nginx Unit刚发布了第一个正式大版本，因此不建议在生产环境直接采用，建议在内网环境或边缘业务上进行测试评估。本文约3000字，预计阅读时间5分钟。

> keywords: Nginx Unit, application server, polyglot, microservices, service mesh

> 关键字：Nginx Unit、应用服务器、多语言支持、微服务、Service Mesh

![Nginx Unit Get Stared](/images/2018/05/nginx-unit-10a96.png)

## 什么是Nginx Unit？

Nginx Unit 1.0是NGINX, Inc.公司在2018年4月12日正式发布的动态Web及应用服务器，Nginx Unit首先定位是应用服务器，也就是application server，对应竞品是unicorn、Tomcat、Passenger等一众动态语言的应用服务器。

Nginx Unit 1.0目前支持的特性有：

* 多语言多版本支持
* 通过REST API动态管理应用（JSON格式），无配置文件
* 配置变更无需重载、不中断服务
* 独立运行或适配微服务架构均可

### 多语言多版本支持

![Nginx Unit can run different languages and language versions in one server](/images/2018/05/nginx-unit-73d12.png)

最令人兴奋的点是Nginx Unit直接支持多语言（Polyglot），并且支持同一语言的不同版本，目前微服务架构盛行，一大特征便是多语言支持，之前可能支持的方式便是不同语言运行在各自的应用服务器下，再通过Nginx进行反向代理，来实现多语言支持，现在可以通过Nginx Unit来直接做应用服务器，而不需要再引入原先的应用服务器，精简了部署架构，目前Nginx Unix支持的语言情况如下：

语言  |  支持版本
--|--
PHP  |  PHP 5, 7
Python  |  Python 2.6, 2.7, 3
Golang  |  Go 1.6 及更新版本
Perl  |  Perl 5.12 及更新版本
Ruby  |  Ruby 2.0 及更新版本
Java  |  计划支持
JavaScript/Node.js  |  计划支持

从上表可以看到，主流的动态语言已经支持了5种，另外计划中支持的Java和Node.js也是很让人期待的发布。

### 动态管理应用

![Architecture of a simple WordPress application
](/images/2018/05/nginx-unit-af634.png)

上图是使用Nginx + Nginx Unit来部署PHP应用的一个典型架构，通过Nginx来做Web服务器（负责静态文件和反向代理服务等），Nginx Unit做应用服务器，这样我们能快速了解Nginx和Nginx Unit的定位区别，Nginx Unit更专注在应用服务器这里。

服务器动态管理能力也是Nginx Unit的亮点，通过Control REST API进行管理，例如对应用的增删改查（JSON格式），在进行管理时，Nginx Unit不需要重载配置，也不会中断服务，这个跟Nginx Unit的架构设计相关，它在设计之初就为动态能力为核心，本文后面会简单介绍Nginx Unit的架构，在此我们先看看Nginx Unit的应用配置管理。

Nginx Unit里面的应用需要配置application和listner两个字段才能提供应用服务，以下是一个最简单的应用服务配置：

```json
{
    "listeners": {
        "*:8300": {
            "application": "blogs"
        }
    },

    "applications": {
        "blogs": {
            "type": "php",
            "processes": 20,
            "root": "/www/blogs/scripts",
            "index": "index.php"
        }
    }
}
```

上面我们定义了一个名为blogs的应用，运行语言是PHP，另外定义了一个监听器，监听在8300端口来提供blogs应用服务，将该配置保存在json文件中，然后通过访问Control API进行应用服务添加，示例如下：

```bash
# curl -X PUT -d @/path/to/blogs.json  \
       --unix-socket /path/to/control.unit.sock http://localhost/config/
```

我们也可以通过如下命令来调用Control API来查看目前的配置：

```bash
# curl --unix-socket /path/to/control.unit.sock http://localhost/config/
# API返回示例如下
{
    "listeners": {
        "*:8300": {
            "application": "blogs"
        }
    },

    "applications": {
        "blogs": {
            "type": "php",
            "user": "nobody",
            "group": "nobody",
            "root": "/www/blogs/scripts",
            "index": "index.php"
        }
    }
}
```

这样使用JSON格式、通过Control REST API进行应用服务管理的方式，使得可以很方便地将基于Nginx Unit的应用服务发布管理融合到持续发布流程中，都不需要对Nginx Unit进行重载或中断服务。

## Nginx Unit的架构

![Nginx Unit Architecture](/images/2018/05/nginx-unit-c9be5.png)

如上图所示，Nginx Unit的进程如下：
* 主进程：唯一进程，负责创建路由、应用进程；
* 路由进程：唯一进程，负责所有出入网络请求的处理，可以快速实施配置变更而不中断服务。另外这个进程是持久的，不会重启；
* 控制器进程：唯一进程，负责接收处理通过管理API过来的配置请求，并且将配置下发给主进程和路由进程；
* 应用进程：每个进程部署在各自沙盒环境中，保证安全隔离，应用进程数量是动态变化的。

Nginx Unit支持按需扩展进程，每个进程都运行在自己的沙箱环境中，这样应用进程才能够在同一个Nginx Unit里面支持多语言多版本的应用了。

## Nginx Unit的未来

从官方介绍中可以得知，接下来Nginx Unit会支持Java和Node.js，这样Nginx Unit就将目前主流Web编程语言都支持了；另外计划支持HTTPS和HTTP/2协议，以及通过URI和主机名进行路由等特性。不难看出这里NGINX, Inc公司的野心，想将Nginx Unit打造成为多语言多版本支持、简易管理及可靠度高的应用服务器，来更好支持微服务、Service Mesh架构。

![NGINX Unit and the NGINX Application Platform](/images/2018/05/nginx-unit-cf44e.png)


目前Nginx Unit刚发布1.0版本，各个语言的支持情况还不会特别完善，例如一些框架在Nginx Unit下运行可能会有问题，抑或是性能表现还需要观察，但总体而言，Nginx Unit还是值得尝试的新应用服务器，从Nginx的生态版本可以看到它的定位还是很清晰的，就是来取代之前各语言的应用服务器，因此我也会持续关注Nginx Unit，最新的技术进展和体会也会持续更新，欢迎关注。

![微信公众号：程序员精进](/images/2018/05/wechat_qr_code.jpg)
