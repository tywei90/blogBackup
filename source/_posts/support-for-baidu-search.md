---
title: 博客搭建系列三：如何使博客支持百度搜索
date: 2016-11-20 23:41:46
tags:
    - 博客
    - 爬虫
    - seo
    - coding.net
---

&emsp;&emsp;我们写自己的博客，当然是想被更多的人看到，分享下自己的研究成果。这里，各种搜索引擎起着重要的作用。查网站的seo，大家可以去[站长之家](http://seo.chinaz.com/), 输入查询的网址即可。搜索引擎是靠爬虫去爬网站的上的内容，我们的博客是一个静态博客，内容托管在github上。打开终端，输入以下命令：
``` bash
curl -A "Mozilla/5.0 (compatible; Baiduspider/2.0; +http://www.baidu.com/search/spider.html)" https://github.com
```
我们会看到结果如下图：

![百度爬虫爬github](/assets/blogImg/curl_baidu.png "百度爬虫爬github")
<!-- more -->
&emsp;&emsp;github把百度爬虫屏蔽了，原因就是百度爬虫爬得太厉害，已经对很多Github用户造成了可用性的问题了。当然，大家也可以尝试下百度爬虫能不能爬到自己的网站内容，我想结果是一样的。我们可以再试试谷歌爬虫，输入以下命令：
``` bash
curl -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://github.com
```
&emsp;&emsp;发现确实可以爬到网页的内容。也就是说，如果用谷歌搜索一些我们博客上的关键字是能搜索到我们文章的，但是百度是搜不到的。国内我们都知道，谷歌被墙了，所以用百度搜索还是有很大比例的。那么我们如何才能使百度搜索到呢？推荐大家一篇技术分析的文章[解决 Github Pages 禁止百度爬虫的方法与可行性分析](http://jerryzou.com/posts/feasibility-of-allowing-baiduSpider-for-Github-Pages/)。

&emsp;&emsp;最好的解决办法还是要将我们博客的内容托管在百度爬虫可以爬的到的地方。最好不需要我们自己购买主机什么的，其实就是一个类似于github的代码托管平台。其实，国内还真有：[coding.net](https://coding.net/)，将gitcafe收购了，是国内最大的代码托管平台，界面也很清爽简洁。操作的步骤其实和github类似。我们这里就不详述了，大家可以参考这篇文章：[解决 Github Pages 禁止百度爬虫的方法2--从gitcafe迁移到coding.net](http://bblove.me/2016/03/06/migrate-pages-from-gitcafe-to-coding/)。

&emsp;&emsp;不过注意下，mac的ssh key获取我们在系列文章一中说过；还有dns的设置大家参考我系列文章二中的设置。这里吐槽一下，github做了cname处理绑定域名后，访问原来提供的github.io域名，会提示301永久重定向到我现在的域名。但是coding.me的域名还是可以正常访问的。这个就有点不爽了。一样的内容，多个域名，会造成网站的分流，seo权重下降。但是服务器是人家的，我们也不能不能做什么。不过，我已经在coding的论坛上提了这个问题，人家技术人员说正在开发，说是下周就能出来。

&emsp;&emsp;这样设置好以后，大家可以用上面的命令看能不能爬到。而且，每次用hexo部署后，我们的博客代码会同时部署到github和coding上，非常方便。这里建议大家将自己的博客目录创建一个github仓库，备份一下里边的source文件夹和_config.yml等配置文件。这样一来我们就不必非得用自己的电脑才能写博客；二来可以做博客内容备份。

&emsp;&emsp;至此，博客搭建系列完毕！








