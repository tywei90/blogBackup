---
title: 用canvas画一轮明月，夜空与流星
date: 2016-11-06 23:56:59
tags: 
    - Canvas
    - Gulp
---

>今天是中秋节，于是突发奇想，欸不如用canvas来画一画月亮吧。

>于是一副用canvas画出的星空就这样诞生了。

##### [Demo](http://ycwalker.com/canvas-moon/)

在这里我用了ES6语法，星星，月亮和流星都单独写成了一个module。

于是我把js一共分成这四个文件：main.js, Moon.js, Stars.js和Meteor.js，后面三个各自export出一个类。
<!-- more -->
##### [源码](https://github.com/ycwalker/canvas-moon)

为了方便，用了gulp做自动化的工具。

#### main.js
``` js
import Stars    from './Stars'
import Moon     from './Moon'
import Meteor   from './Meteor'

let canvas = document.getElementById('canvas'),
    ctx = canvas.getContext('2d'),
    width = window.innerWidth,
    height = window.innerHeight,
    //实例化月亮和星星。流星是随机时间生成，所以只初始化数组
    moon = new Moon(ctx, width, height),
    stars = new Stars(ctx, width, height, 200),
    meteors = [],
    count = 0

canvas.width = width
canvas.height = height

//流星生成函数
const meteorGenerator = ()=> {
    //x位置偏移，以免经过月亮
    let x = Math.random() * width + 800
    meteors.push(new Meteor(ctx, x, height))

    //每隔随机时间，生成新流星
    setTimeout(()=> {
        meteorGenerator()
    }, Math.random() * 2000)
}

//每一帧动画生成函数
const frame = ()=> {
    //每隔10帧星星闪烁一次，节省计算资源
    count++
    count % 10 == 0 && stars.blink()

    moon.draw()
    stars.draw()

    meteors.forEach((meteor, index, arr)=> {
        //如果流星离开视野之内，销毁流星实例，回收内存
        if (meteor.flow()) {
            meteor.draw()
        } else {
            arr.splice(index, 1)
        }
    })
    requestAnimationFrame(frame)
}

meteorGenerator()
frame()
```

开头分别引入了另外三个module，分别是星星，月亮和流星。

接着初始化了月亮和星星，但由于流星是不定时随机生成的，所以初始化一个数组用来保存接下来生成的流星。

在每一帧中，分别调用moon，star和meteor的draw函数，用来画出每一帧，特别的，因为星星需要闪烁，流星需要移动，所以在draw之前对半径和坐标进行处理。如果流星跑出了canvas外，就从数组中清除相应的流星，从而解除引用和回收内存。
#### Moon.js
``` js
export default class Moon {
    constructor(ctx, width, height) {
        this.ctx = ctx
        this.width = width
        this.height = height
    }

    draw() {
        let ctx = this.ctx,
            gradient = ctx.createRadialGradient(
            200, 200, 80, 200, 200, 800)
        //径向渐变
        gradient.addColorStop(0, 'rgb(255,255,255)')
        gradient.addColorStop(0.01, 'rgb(70,70,80)')
        gradient.addColorStop(0.2, 'rgb(40,40,50)')
        gradient.addColorStop(0.4, 'rgb(20,20,30)')
        gradient.addColorStop(1, 'rgb(0,0,10)')
        ctx.save()
        ctx.fillStyle = gradient
        ctx.fillRect(0, 0, this.width, this.height)
        ctx.restore()
    }
}
```
这是月亮的类，主要用到了canvas里的径向渐变效果。为了达到和谐的程度，我试了好久T_T...
#### Stars.js
``` js
export default class Stars {
    constructor(ctx, width, height, amount) {
        this.ctx = ctx
        this.width = width
        this.height = height
        this.stars = this.getStars(amount)
    }

    getStars(amount) {
        let stars = []
        while (amount--) {
            stars.push({
                x: Math.random() * this.width,
                y: Math.random() * this.height,
                r: Math.random() + 0.2
            })
        }
        return stars
    }

    draw() {
        let ctx = this.ctx
        ctx.save()
        ctx.fillStyle = 'white'
        this.stars.forEach(star=> {
            ctx.beginPath()
            ctx.arc(star.x, star.y, star.r, 0, 2 * Math.PI)
            ctx.fill()
        })
        ctx.restore()
    }

    //闪烁，星星半径每隔10帧随机变大或变小
    blink() {
        this.stars = this.stars.map(star=> {
            let sign = Math.random() > 0.5 ? 1 : -1
            star.r += sign * 0.2
            if (star.r < 0) {
                star.r = -star.r
            } else if (star.r > 1) {
                star.r -= 0.2
            }
            return star
        })

    }
}
```
星星的集合。因为不至于给每一个星星都写成单独的对象，于是就写了一个星星的集合类，所有的星星都保存在实例的stars中。其中的blink函数用来随机改变每一个星星的半径大小，从而产生闪烁的效果。
#### Meteor.js
``` js
export default class Meteor {
    constructor(ctx, x, h) {
        this.ctx = ctx
        this.x = x
        this.y = 0
        this.h = h
        this.vx = -(4 + Math.random() * 4)
        this.vy = -this.vx
        this.len = Math.random() * 300 + 500
    }

    flow() {
        //判定流星出界
        if (this.x < -this.len || this.y > this.h + this.len) {
            return false
        }
        this.x += this.vx
        this.y += this.vy
        return true
    }

    draw() {
        let ctx = this.ctx,
            //径向渐变，从流星头尾圆心，半径越大，透明度越高
            gra = ctx.createRadialGradient(
                this.x, this.y, 0, this.x, this.y, this.len)

        const PI = Math.PI
        gra.addColorStop(0, 'rgba(255,255,255,1)')
        gra.addColorStop(1, 'rgba(0,0,0,0)')
        ctx.save()
        ctx.fillStyle = gra
        ctx.beginPath()
        //流星头，二分之一圆
        ctx.arc(this.x, this.y, 1, PI / 4, 5 * PI / 4)
        //绘制流星尾，三角形
        ctx.lineTo(this.x + this.len, this.y - this.len)
        ctx.closePath()
        ctx.fill()
        ctx.restore()
    }
}
```
流星就比较有意思啦。猜猜每一个流星是怎么画的？

实际上每一个流星的轮廓由一个半圆和一个三角形组成，类似于一个不倒翁。然后整体倾角45度，并且填充时用上一个径向渐变，就可以相当完美的达到流行尾巴那样渐行渐远渐模糊的样子。

对，就是这么干净利落~

最后看了一下CPU和GPU的占用，还好，优化的还比较到位，我那渣族手机都能跑的很流畅...

今天是中秋节，可惜我这下雨了...没月亮可看...

不过我有了这个月亮。

“但愿人长久，千里共婵娟”，千里之外的朋友，看到同一轮“明月”，也是缘分吧~