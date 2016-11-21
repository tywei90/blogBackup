---
title: 博客搭建系列二：个人博客绑定自己域名
date: 2016-11-15 22:47:46
tags:
    - 博客
    - 域名
---

&emsp;&emsp;上一篇文章，我们详细说明了如何用hexo搭建个人博客，并且有了自己的博客地址。但是，有的同学可能觉得还不够牛逼。ok，那下面我分享下自己的博客是如何绑定自己申请的域名。

## 1、申请域名
&emsp;&emsp;博客绑定域名，首先，你得有个域名。关于申请域名的网址有很多，国内有万网（被阿里云收购了），新网等。不过我不推荐在国内购买域名，需要备案等一系列手续非常麻烦。

&emsp;&emsp;国外的域名注册商很多，用的比较多的有GoDaddy、namecheap、name.com等，至于选哪个，推荐大家一篇知乎上的文章 https://www.zhihu.com/question/19551906 没错，我是在namecheap上注册的，现在貌似没有优惠码了，不过他们家免费送Whois 隐私保护（Whois查询不到注册人），SSL证书不再免费，不过和域名一起购买只需再花2$，总共一年也就70几块钱。
<!-- more -->
&emsp;&emsp;这里推荐大家申请后缀为.com的域名。当然了，如果你喜欢有个性的域名，或者希望网址能短一点，选其他的应该更容易命中。还有点要注意，namecheap不支持支付宝，银联，大家可以选择带有master或者visa标志的银行卡支付。这里再给大家推荐个比价网站 https://www.domcomp.com/ 有个网站第一年很便宜，后面续约越来越贵。我想一般我们的域名申请了应该会用挺久的吧。

## 2、注册[DNSPod](https://www.dnspod.cn)，添加域名和记录

&emsp;&emsp;DNS域名解析一般都是用的DNSPod，大家注册以后去域名解析一栏添加自己刚申请的域名。然后设置如下图所示:

![DNS域名解析设置](/assets/blogImg/dnspod.png "DNS域名解析设置")

&emsp;&emsp;大家只需要添加红框部分内容即可，其他设置下一篇文章会说。这里我们添加了主机记录分别为www和@两种类型，分别对应着您的网址带www和不带www的映射。防止有的浏览器默认添加www导致网页打不开。

&emsp;&emsp;一定要注意我们的CNAME记录指向值，我看网上很多都是设置的一个ip值，其实这样不好。因为第一，ip地址可能会变。第二，写死ip地址，万一这个主机挂了，或者某地区的这个ip网络信号不好，我们的网页就不打不开。大家可以在命令行dig一下，上一篇文章说过这个命令。

&emsp;&emsp;还有个非常重要没说，就是去你域名注册的网站去绑定DNSPod的dns服务器设置。如果是namecheap网站，替换掉网址 https://ap.www.namecheap.com/domains/domaincontrolpanel/your_domain_name/domain 中的your_domain_name，添加两条NAMESERVERS记录，设置如下图所示。

![绑定DNSPod服务器设置](/assets/blogImg/dns.png "绑定DNSPod服务器设置")

&emsp;&emsp;这样，你的网站才能被DNSPod做dns解析。一般需要一定时间才能生效，我当时是半个小时左右吧。

## 3、添加CNAME文件
&emsp;&emsp;在你的博客source文件夹里创建CNAME文件，不带任何后缀，里面添加你的域名信息，如：weitianyao.com（注意前面不添加`http://`），如下图：

![添加CNAME文件](/assets/blogImg/CNAME.png "添加CNAME文件")

&emsp;&emsp;然后就`hexo d`试试吧。





