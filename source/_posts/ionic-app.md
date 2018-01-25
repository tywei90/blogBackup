---
title: 基于ionic的混合APP实战
date: 2018-01-16 11:36:54
tags:
    - ionic
    - phonegap
    - 混合APP
reward: true
toc: true
---

这个项目做得比较早，当时是基于ionic1和angular1做的。做了四个tabs的app，首页模仿携程首页，第二页主要是phonegap调用手机核心功能，第三页模仿微信和qq聊天页，第四页模仿一般手机的表单设置页。同时还模仿知乎做了一个侧边栏页(账号：wty，密码：123456)。

没有后台，纯前端展示，功能还比较多，调用系统的声音、震动和手机设备信息等。有二维码扫描功能，还做了类似qq消息可拖拽效果，上拉下拉刷新，轮播图组件。

当时做的ppt下载: [2016.2.3技术分享ionic实战.ppt](/assets/ppt/ionic_app.ppt)

[安卓apk下载](/assets/apk/ionic_app.apk)  

![安卓apk下载二维码](/assets/img/apk.png "安卓apk下载二维码")

欢迎大家star学习交流：[线上地址](https://ionic.wty90.com/)&emsp;[github地址](https://github.com/tywei90/ionic_app_production)

<!-- more -->
## 一、基本概念

### 1. Angularjs简介
Angularjs是一款优秀的前端 JS 框架，已用于 Google 的多款产品当中 如 Gmail、Maps、Calender 等。AngularJS有着诸多特性，最为核心的是：MVVM、模块化、自动化双向数据绑定、语义标签、依赖注入，等等。

### 2. Ionic简介
Ionic是一个强大的 HTML5 应用程序开发框架，具有速度快，界面现代化、美观等特点。特别适合用于基于 Hybird 模式的 HTML5 移动应用程序开发。

### 3. Phonegap简介
Phonegap是一个用基于 HTML， CSS 和 JavaScript 的，创建移动跨平台移动应用程序的
快速开发平台。它使开发者能够手机的核心功能——包括地理定位，加速器，联系人，声音和振动等，此外PhoneGap 拥有丰富的插件，可以调用。

## 二、项目各tab主要功能介绍

### 1. 初始化配置
* 手机上app显示的图标、名称、开机画面
* 注入依赖
* 隐藏显示键盘
* hammer触屏手势插件配置
* 菜单栏的位置、导航条文字位置、回退按钮图标等
* 切换页面的过渡效果（bug）
* AngularUI Router
* services服务

### 2. tab-home
* 幻灯指令 ion-slide-box
* 触屏手势切换页面
* 栅格系统
* 触屏手势touch-bases和hammerjs
* ng-init、ng-click、 ng-src、 ng-repeat指令，双向数据绑定
* 打开app内置的浏览器webview方法
* 上拉刷新

### 3. tab-dash
* phonegap功能的应用：二维码扫描、调用系统弹窗、震动铃声功能、获取设备信息
* ion-side-menus侧边栏功能
* ionic 动态组件 $ionicModal弹出登录界面
* ng-show、ng-model 双向数据绑定实现登录验证的实时监控
* ionic 动态组件 $ionicPopup弹出注销界面
* 更换头像（访问手机摄像头、图库功能）
* 切换主题颜色

### 4. tab-chats
* 删除按钮和重新排序按钮
* 下拉刷新
* 滑动显示分享编辑按钮
* 长按显示动态组件$ionicActionSheet选项
* 红色消息badge

### 5. tab-account
* ionic的表单应用
* “声音”选项被选中播放铃声
* “震动”选项被选中开始震动
* 实现全选、全不选、反选的功能
* ionic动态组件$ionicPopup
* 根据被选择数显示相应弹窗内容

## 三、演示如下：

![ionic实战动态图演示](/assets/img/ionic_app.gif)  


## 四、总 结
优点： 通过使用 web 技术开发 App，采用 Cordova/PhoneGap之类进行打包封装。优点是采用标准的web技术开发，避免了不同平台原生开发体系的学习，学习成本低， 上手快、 效率高，一次开发微信 wap app 全部搞定；

缺点：app 在 android 平台性能上有一些损失， 但是相信硬件的发展会接近原生。


## 参考文献
1. [PhoneGap3.4安装视频教程下载](http://bbs.phonegap100.com/thread-668-1-1.html)
3. [Angular1官网](http://www.angularjs.org/)
5. [Angular中文社区](http://www.angularjs.cn/)
7. [AngularJS Nice Things](http://www.ngnice.com/)
8. [phonegap 中文网](http://www.phonegap100.com)
9. [ionic官网](http://ionicframework.com/)
