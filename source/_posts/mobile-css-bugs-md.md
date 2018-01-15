---
title: 移动端bug总结和一些奇淫技巧
date: 2016-11-27 23:25:15
tags:
    - 移动端
    - css
    - 兼容性
    - bug
    - 奇淫技巧
reward: true
toc: true
---

这里总结下我平时自己遇到和在网上看到的一些移动端bug，希望对大家有用。

## part1
1、uc浏览器的flexbox兼容性bug，在父元素上应用flex属性时，直接子元素要`display: block`，否则没有效果。
``` less
//兼容uc的 space-around
div{
    display: flex;
    justify-content: space-around;
    font-size: 48/@brem;
    color: white;
    span{
        letter-spacing: 5/@brem;
    }
    //兼容UC浏览器
    >*{
        display: block;
    }
}
```

<!-- more -->

2、android上的uc浏览器父元素`display:flex`，子元素margin:auto水平垂直居中没效果，加上`justify-content`和`align-items: center`即可，在低端android版本上需要再加上`text-align: center`才能水平居中。

``` html
<div>
    <p>wap端水平垂直居中</p>
</div>
```

``` less
div {
    display: flex; 
    //防止移动端bug
    justify-content: center;
    align-items: center;
    text-align: center;
}
p {margin: auto;}
```

3、苹果机click事件代理到body，document上会不触发 

4、pc端垂直居中
父元素`display: table` 并且显示定义高
子元素 `display: table-cell; vertical-align: middle`

5、em是相对于元素自身的font-size计算的，可以在局部应用

6、ios手机和android上的uc浏览器等在页面滑动时会禁止一切js脚本，保存为一个状态A，滑动结束后才会触发scroll事件，从状态A开始。所以不能使用setTimeout进行倒计时。可以使用new Date计算时间差。

7、`background-size`属性在IE9下不支持，背景图片默认是显示图片自身大小的

8、`padding-top/padding-left`百分比都是相对元素本身的宽度

9、对于小图片可以用base64编码节省http请求
http://www.cnblogs.com/coco1s/p/4375774.html

## part2
1、最痛恨的是红米手机，ua返回iphone，需要结合platform判断，但是还不准确，导致需要ios和android区别对待的时候就坑了。

2、是fixed的问题。这个解决办法是尽量不要用，不过ios7及以下才会出现这个问题。某些情况下红米也会有这个问题。（最近刚刚遇到，已经被坑挂了）。

3、如果你想要使用css3的动画，那么一定要变着方式使用3d gpu加速的方式，不要试着left，height，width这样的元素进行变换了，android4.4以下版本卡死你。

4、ios全线点击会有300毫秒延迟，使用fastclick解决。这个插件最良心了。

5、web app像素眼设计会纠结你1px边框问题。解决办法有相应知乎大牛答过。

6、qq浏览，uc浏览以及ios的浏览器，滚动时不会触发scroll事件，但会触发touchmove。当停止滚动后会出发scroll。

7、滚动有iscroll插件，但是还是使用原生的比较好。

8、meta功能要用好，禁止缩放，缩放比例，屏蔽电话号码等功能很实用。（手机回答就不列举了）。

9、如果想要像手机淘宝那样的各个平台看起来展示效果一致，那么就使用rem来做单位。

10、-webkit-tap-highlight-color可以取消点击高亮。

11、localStorage在浏览器开启无痕模式下ios会抛异常，导致js中断。

12、一些情况下对非可点击元素监听click事件，ios下不会触发，css增加cursor:pointer就搞定了。当然想要干脆静止点击就是not-allowed。

13、android4.4以下版本，设置圆角属性需要在直接元素上，向父元素设置圆角并且指定overflow:hidden是不会生效的。

## part3
1、图片大小最好是标注的2倍，这样可以保证在各类机型上的清晰

2、小图片最好拼合起来做成雪碧图，然后可以在TinyPNG – Compress PNG images while preserving transparency网站上压缩后再Base64嵌入html或css中。移动端要尽可能减少请求。不过太大的图片就不要base64了，性能会下降。一般以10k为界

3、rem方案相当复杂，存在非常多的兼容问题，以至于阿里这边还专门有一个Flexible库来解决这一系列问题。但是兼容性问题解决后开发会变得非常畅快。

4、图片比较多的页面务必要做懒加载（也就是滚动到图片的时候才加载）

5、border-radius不要随便乱用，在很多安卓机型上都会出现锯齿，非常丑

6、css动画请使用transform，而非直接控制width,height,margin，否则会造成大量的计算，性能堪忧。transfrom会把元素独立出来单独计算的。

7、如果动画比较多或者面积比较大，想提高性能的话可以用transform3D，就算不设置3d变换也会促使浏览器开启硬件加速

8、flex布局挺好，但有点兼容性问题，需要同时写好几个带前缀的私有属性才能保证大多数机型的适配。推荐使用autoprefixer，不过要记得自己定制兼容浏览器列表，不然会有很多-ms-,-o-这类的属性，在移动端是没有用的 