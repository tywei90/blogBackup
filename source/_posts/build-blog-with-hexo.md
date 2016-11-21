---
title: 博客搭建系列一：用hexo搭建个人博客
date: 2016-11-12 18:04:23
tags:
    - hexo
    - 博客
---

&emsp;&emsp;本来不准备写的，因为网上一搜一大堆讲如何用hexo搭建博客的文章，但是，一来这个markdown语法以前没写过，想用来练练手。二来，网上文章有的写的比较早，一些配置和api可能已经变了。好的，啰嗦玩了，下面开始正文。

&emsp;&emsp;首先声明，本教程是针对mac的，不保证windows执行没有问题

## 1、配置环境
> 安装node

&emsp;&emsp;nodejs是服务器语言，借助google的chrome浏览器V8引擎，可以让前端js脚本运行在服务器端，前后端语言统一，不要太美~在这里，他主要是用来生成静态页面的。[Node.js官网](https://nodejs.org/en/)下载相应平台的最新版本，一路安装即可。
<!-- more -->
> 安装git

&emsp;&emsp;把本地的hexo内容提交到github上去，安装Xcode就自带有Git。可以用命令行，当然你也可以source tree可视化工具来管理。
> 申请github账号

&emsp;&emsp;hexo博客是一个静态博客，内容是托管在github上的。去官网注册申请，然后配置下SSH Keys，这样就不用每次提交都输入用户名和密码了。[mac ssh key 获取](http://blog.csdn.net/yhqbsand/article/details/22763411)

## 2、hexo搭建博客
> 全局安装hexo

&emsp;&emsp;确保上述环境安装好之后，全局安装hexo:
```
sudo npm install -g hexo
```
> 初始化

&emsp;&emsp;创建项目文件夹，如myBlog
```
mkdir myBlog
cd myBlog
hexo init
```
&emsp;&emsp;这样，hexo就安装完毕了

> 生成静态页面

&emsp;&emsp;在myBlog文件夹下，执行
``` 
hexo g (或hexo generate)
```
&emsp;&emsp;这样，hexo就会编译生成静态页面，在public目录下
> 启动本地服务器

``` 
hexo s (或hexo server)
```
&emsp;&emsp;在bash命令行，按下command键单击 http://localhost:4000/ ，即可用浏览器打开此页面。可以做本地预览
&emsp;&emsp;恭喜！你已经看到自己的博客了。但是域名和服务器都是自己电脑，我们需要关联github，继续往下看

## 3、部署github
> 新建仓库

&emsp;&emsp;在github上创建新的仓库，仓库名必须为[your_user_name.github.io]，将自己的github用户名替换掉your_user_name。
> 编辑文件_config.yml，建立关联

&emsp;&emsp;在myBlog根目录找到_config.yml文件，打开它，如果你有sublime编辑器，并安装全局命令。可直接
```
subl -w _config.yml
```
&emsp;&emsp;在最下面，改成这样。替换掉your_user_name。一定要注意: 这里的所有配置:后面都要加空格
```
deploy: 
  type: git
  repository: https://github.com/your_user_name/your_user_name.github.io.git
  branch: master
```
> npm安装依赖，才能使用git部署

```
npm install hexo-deployer-git --save
```
> 将博客部署到github

```
hexo d (或hexo deploy)
```
&emsp;&emsp;打开网址 http://tywei90.github.io/ tywei90是我的github用户名，换成你自己的就行。看到没？你的博客已经上线了~

&emsp;&emsp;等等。。好像哪里不对。如果这样，岂不是每个github用户都有一个自己的域名，github那来的这么多域名？其实你只要在你的bash命令行执行命令如下：
```
dig tywei90.github.io
```
我们会看到：
![dig结果](/assets/blogImg/dig.jpg "dig结果")

&emsp;&emsp;dig命令是查网址的dns解析的，我们发现博客地址被CNAME到github.map.fastly.net.上，他的服务器ip是151.101.100.133。什么意思呢，我们先来解释下CNAME。

&emsp;&emsp;CNAME指别名记录也被称为规范名字。这种记录允许您将多个域名需要指向同一服务器IP，此时您就可以将一个域名做A记录指向服务器IP，然后将其他的域名做别名(即CNAME)到A记录的域名上；那么当您的服务器IP地址变更时，您就可以不必对一个一个域名做更改指向了，只需要更改A记录的那个域名到服务器新IP上，其他做别名（即CNAME）的那些域名的指向将自动更改到新的IP地址上。

&emsp;&emsp;总结下：也就是说，我们的博客地址都会被映射到ip为151.101.100.133的主机上，然后github会根据我们的用户名查找相应的静态文件，然后返回。

## 4、相关知识
> hexo部署三步走

每次部署博客都要执行下面三步
```
hexo clean
hexo g
hexo d
```
> hexo常用命令

```
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
hexo help  #查看帮助
hexo version  #查看Hexo的版本
```
> hexo主题

hexo有很多漂亮的主题可选，这也是为什么我没有选择jekyll的原因。
* [Yilia](https://github.com/litten/hexo-theme-yilia) - Responsive and simple style 强烈推荐，我用得就是这个。(ps: 作者人也很nice~)
* [NexT](http://theme-next.iissnan.com/) -Elegant Theme for Hexo 都有自己的官网了，用的人很多
* [Cover](https://github.com/daisygao/hexo-themes-cover) - A chic theme with facebook-like cover photo 

至于主题如何配置，我就不细说了，不同主题不一样，大家可以去参阅相关文档。
> 博客写作技巧

1、如何让文章想只显示一部分和一个 阅读全文 的按钮？ 
答：在文章中加一个 ` <!--more--> ， <!--more--> ` 后面的内容就不会显示出来了。

2、如何给文章添加标签？
答：在文章的开头有个tags配置项，配置格式如下：
```
tags:
    - hexo
    - 博客
```











