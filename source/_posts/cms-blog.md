---
title: 一个基于Vue.js+Mongodb+Node.js的博客内容管理系统
date: 2018-02-16 21:28:41
tags:    
    - cms
    - 博客
reward: true
toc: true
---
这个项目最初其实是fork别人的项目。当初想接触下mongodb数据库，找个例子学习下，后来改着改着就面目全非了。后台和数据库重构，前端增加了登录注册功能，仅保留了博客设置页面，但是也优化了。

欢迎大家star学习交流：[线上地址](https://cms.wty90.com/)&emsp;[github地址](https://github.com/tywei90/CMS-of-Blog_Production)

<!-- more -->
## 一、功能特点
1. 一个基本的博客内容管理器功能，如发布并管理文章等
2. 每个用户可以通过注册拥有自己的博客
3. 支持markdown语法编辑
4. 支持代码高亮
5. 可以管理博客页面的链接
6. 博客页面对移动端适配优化
7. 账户管理(修改密码)
8. 页面足够大气、酷炫嘿

## 二、用到的技术和实现思路：

### 2.1 前端：Vue全家桶
* Vue.js
* Vue-Cli
* Vue-Resource
* Vue-Validator
* Vue-Router
* Vuex
* Vue-loader

### 2.2 后端
* Node.js
* mongoDB (mongoose)
* Express

### 2.3 工具和语言
* Webpack
* ES6
* SASS
* Jade

### 2.4 整体思路：
* Node服务端除了主页和首页外，不做模板渲染，渲染交给浏览器完成
* Node服务端不做任何路由切换的内容，这部分交给Vue-Router完成
* Node服务端只用来接收请求，查询数据库并用来返回值

所以这样做前后端几乎完全解耦，只要约定好restful风格的数据接口，和数据存取格式就OK啦。

后端我用了mongoDB做数据库，并在Express中通过mongoose操作mongoDB，省去了复杂的命令行，通过Javascript操作无疑方便了很多。


## 三、更新内容

在原来项目的基础上，做了如下更新：
1. 数据库重新设计，改成以用户分组的subDocs数据库结构
2. 应数据库改动，所有接口重新设计，并统一采用和网易[立马理财](https://www.lmlc.com/)一致的接口风格
3. 删除原来游客模式，增加登录注册功能，支持弹窗登录。
4. 增加首页，展示最新发布文章和注册用户
5. 增加修改密码，登出，注销等功能。
6. 优化pop弹窗组件，更加智能，更多配置项，接近网易$.dialog组件。并且一套代码仅修改了下css，实现相同接口下pc端弹窗和wap端toast功能。
7. 增加移动端适配
8. 优化原来代码，修复部分bug。

更多的更新内容请移步项目[CMS-of-Blog_Production](https://github.com/tywei90/CMS-of-Blog_Production/commits/master)和[CMS-of-Blog](https://github.com/tywei90/CMS-of-Blog/commits/master)。

## 四、核心代码分析
原作者也写过分析的[文章](https://ycwalker.com/2016/09/03/blog-cms/)。这里，主要分析一下我更新的部分。
### 4.1. 数据库
对原数据库进行重新设计，改成以用户分组的subDocs数据库结构。这样以用户为一个整体的数据库结构更加清晰，同时也更方便操作和读取。代码如下：
```js
var mongoose =  require('mongoose'),
    Schema =    mongoose.Schema

    articleSchema = new Schema({
        title: String,
        date: Date,
        content: String,
    }),

    linkSchema = new Schema({
        name: String,
        href: String,
        newPage: Boolean
    }),

    userSchema = new Schema({
        name: String,
        password: String,
        email: String,
        emailCode: String,
        createdTime: Number,
        articles: [articleSchema],
        links: [linkSchema]
    }),

    User = mongoose.model('User', userSchema);

mongoose.connect('mongodb://localhost/platform')
mongoose.set('debug', true)

var db = mongoose.connection
db.on('error', function () {
    console.log('db error'.error)
})
db.once('open', function () {
    console.log('db opened'.silly)
})

module.exports = {
    User: User
}
```

代码一开始新定义了三个Schema：articleSchema、linkSchema和userSchema。而userSchema里又嵌套了articleSchema和linkSchema，构成了以用户分组的subDocs数据库结构。Schema是一种以文件形式存储的数据库模型骨架，不具备数据库的操作能力。然后将将该Schema发布为Model。Model由Schema发布生成的模型，具有抽象属性和行为的数据库操作对。由Model可以创建的实体，比如新注册一个用户就会创建一个实体。

数据库创建了之后需要去读取和操作，可以看下注册时发送邮箱验证码的这段代码感受下。
```js
router.post('/genEmailCode', function(req, res, next) {
    var email = req.body.email,
    resBody = {
        retcode: '',
        retdesc: '',
        data: {}
    }
    if(!email){
        resBody = {
            retcode: 400,
            retdesc: '参数错误',
        }
        res.send(resBody)
        return
    }
    function genRandomCode(){
        var arrNum = [];
        for(var i=0; i<6; i++){
            var tmpCode = Math.floor(Math.random() * 9);
            arrNum.push(tmpCode);
        }
        return arrNum.join('')
    }
    db.User.findOne({ email: email }, function(err, doc) {
        if (err) {
            return console.log(err)
        } else if (doc && doc.name !== 'tmp') {
            resBody = {
                retcode: 400,
                retdesc: '该邮箱已注册',
            }
            res.send(resBody)
        } else if(!doc){  // 第一次点击获取验证码
            var emailCode = genRandomCode();
            var createdTime = Date.now();
            // setup e-mail data with unicode symbols
            var mailOptions = {
                from: '"CMS-of-Blog 👥" <tywei90@163.com>', // sender address
                to: email, // list of receivers
                subject: '亲爱的用户' + email, // Subject line
                text: 'Hello world 🐴', // plaintext body
                html: [
                    '<p>您好！恭喜您注册成为CMS-of-Blog博客用户。</p>',
                    '<p>这是一封发送验证码的注册认证邮件，请复制一下验证码填写到注册页面以完成注册。</p>',
                    '<p>本次验证码为：' + emailCode + '</p>',
                    '<p>上述验证码30分钟内有效。如果验证码失效，请您登录网站<a href="https://cms.wty90.com/#!/register">CMS-of-Blog博客注册</a>重新申请认证。</p>',
                    '<p>感谢您注册成为CMS-of-Blog博客用户！</p><br/>',
                    '<p>CMS-of-Blog开发团队</p>',
                    '<p>'+ (new Date()).toLocaleString() + '</p>'
                ].join('') // html body
            };
            // send mail with defined transport object
            transporter.sendMail(mailOptions, function(error, info){
                if(error){
                    return console.log(error);
                }
                // console.log('Message sent: ' + info.response);
                new db.User({
                    name: 'tmp',
                    password: '0000',
                    email: email,
                    emailCode: emailCode,
                    createdTime: createdTime,
                    articles: [],
                    links: []
                }).save(function(err) {
                    if (err) return console.log(err)
                    // 半小时内如果不注册成功，则在数据库中删除这条数据，也就是说验证码会失效
                    setTimeout(function(){
                        db.User.findOne({ email: email }, function(err, doc) {
                            if (err) {
                                return console.log(err)
                            } else if (doc && doc.createdTime === createdTime) {
                                db.User.remove({ email: email }, function(err) {
                                    if (err) {
                                        return console.log(err)
                                    }
                                })
                            }
                        })
                    }, 30*60*1000);
                    resBody = {
                        retcode: 200,
                        retdesc: ''
                    }
                    res.send(resBody)
                })
            });
        }else if(doc && doc.name === 'tmp'){
            // 在邮箱验证码有效的时间内，再次点击获取验证码(类似省略)
            ...
        }
    })
})
```
后台接受到发送邮箱验证码的请求后，会初始化一个tmp的用户。通过`new db.User()`会创建一个User的实例，然后执行`save()`操作会将这条数据写到数据库里。如果在半小时内没有注册成功，通过匹配邮箱，然后`db.User.remove()`将这条数据删除。更多具体用法请移步[官方文档](https://docs.mongodb.com/manual/tutorial/getting-started/)。

### 4.2. 后台
将所有请求分为三种：
* ajax异步请求，统一路径：`/web/`
* 公共页面部分，如博客首页、登录、注册等，统一路径：`/`
* 与博客用户id相关的博客部分，统一路径：`/:id/`  

这样每个用户都可以拥有自己的博客页面，具体代码如下：
```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var routes = require('./index');
var db = require('./db')
var app = express();

// view engine setup
app.set('views', path.join(__dirname, '../'));
app.set('view engine', 'jade');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use('/public',express.static(path.join(__dirname, '../public')));

// 公共ajax接口(index.js)
app.use('/web', routes);

// 公共html页面，比如登录页，注册页
app.get('/', function(req, res, next) {
    res.render('common', { title: 'CMS-blog' });
})

// 跟用户相关的博客页面(路由的第一个参数只匹配与处理的相关的，不越权！)
app.get(/^\/[a-z]{1}[a-z0-9_]{3,15}$/, function(req, res, next) {
    // format获取请求的path参数
    var pathPara = req._parsedUrl.pathname.slice(1).toLocaleLowerCase()
    // 查询是否对应有相应的username
    db.User.count({name: pathPara}, function(err, num) {
        if (err) return console.log(err)
        if(num > 0){
            res.render('main', { title: 'CMS-blog' });
        }else{
            // 自定义错误处理
            res.status(403);
            res.render('error', {
                message: '该用户尚未开通博客。<a href="/#!/register">去注册</a>',
            });
        }
    })
})

// catch 404 and forward to error handler
app.use(function(req, res, next) {
    var err = new Error('Not Found');
    err.status = 404;
    next(err);
});

// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
    app.use(function(err, req, res, next) {
        res.status(err.status || 500);
        res.render('error', {
            message: err.message,
            error: err
        });
    });
}

module.exports = app;
```
具体的ajax接口代码大家可以看server文件夹下的index.js文件。

### 4.3. pop/toast组件

在原项目基础上，优化了pop弹窗组件，更加智能，更多配置项，接近网易$.dialog组件。使并且一套代码仅修改了下css，实现相同接口下pc端弹窗和wap端toast功能。因为有部分格式化参数代码在vuex的action里，有时间，可以将这个进一步整理成一个vue组件，方便大家使用。

#### 4.3.1 pop/toast组件配置参数说明
* `pop`: 弹窗的显示与否, 根据content参数，有内容则为true
* `css`: 自定义弹窗的class, 默认为空
* `showClose`: 为false则不显示关闭按钮, 默认显示
* `closeFn`: 弹窗点击关闭按钮之后的回调
* `title`: 弹窗的标题，默认'温馨提示', 如果不想显示title, 直接传空
* `content`（required）: 弹窗的内容，支持传html
* `btn1`: '按钮1文案|按钮1样式class', 格式化后为btn1Text和btn1Css
* `cb1`: 按钮1点击之后的回调，如果cb1没有明确返回true，则默认按钮点击后关闭弹窗
* `btn2`: '按钮2文案|按钮2样式class', 格式化后为btn2Text和btn2Css
* `cb2`: 按钮2点击之后的回调，如果cb2没有明确返回true，则默认按钮点击后关闭弹窗。按钮参数不传，文案默认'我知道了'，点击关闭弹窗
* `init`: 弹窗建立后的初始化函数，可以用来处理复杂交互（注意弹窗一定要是从pop为false变成true才会执行）
* `destroy`: 弹窗消失之后的回调函数
* `wapGoDialog`: 在移动端时，要不要走弹窗，默认false，走toast

#### 4.3.2 pop/toast组件代码

**模板**

```html
<template>
    <div class="m-dialog" :class="getPopPara.css">
        <div class="dialog-wrap">
            <span class="close" @click="handleClose" v-if="getPopPara.showClose">+</span>
            <div class="title" v-if="getPopPara.title">{{getPopPara.title}}</div>
            <div class="content">{{{getPopPara.content}}}</div>
            <div class="button">
                <p class="btn" :class="getPopPara.btn1Css" @click="fn1">
                    <span>{{getPopPara.btn1Text}}</span>
                </p>
                <p class="btn" :class="getPopPara.btn2Css" @click="fn2" v-if="getPopPara.btn2Text">
                    <span>{{getPopPara.btn2Text}}</span>
                </p>
            </div>
        </div>
    </div>
</template>
```

**脚本**

```js
import {pop}                from '../vuex/actions'
import {getPopPara}         from '../vuex/getters'
import $                    from '../js/jquery.min'

export default{
    computed:{
        showDialog(){
            return this.getPopPara.pop
        }
    },
    vuex: {
        getters: {
            getPopPara
        },
        actions: {
            pop
        }
    },
    methods: {
        fn1(){
            let fn = this.getPopPara.cb1
            let closePop = false
            //  如果cb1函数没有明确返回true，则默认按钮点击后关闭弹窗
            if(typeof fn == 'function'){
                closePop = fn()
            }
            // 初始值为false, 所以没传也默认关闭
            if(!closePop){
                this.pop()
            }
            // !fn && this.pop()
        },
        fn2(){
            let fn = this.getPopPara.cb2
            let closePop = false
            //  如果cb1函数没有明确返回true，则默认按钮点击后关闭弹窗
            if(typeof fn == 'function'){
                closePop = fn()
            }
            // 初始值为false, 所以没传也默认关闭
            if(!closePop){
                this.pop()
            }
            // !fn && this.pop()
        },
        handleClose(){
            // this.pop()要放在最后，因为先执行所有参数就都变了
            let fn = this.getPopPara.closeFn
            typeof fn == 'function' && fn()
            this.pop()
        }
    },
    watch:{
        'showDialog': function(newVal, oldVal){
            // 弹窗打开时
            if(newVal){
                // 增加弹窗支持键盘操作
                $(document).bind('keydown', (event)=>{
                    // 回车键执行fn1，会出现反复弹窗bug
                    if(event.keyCode === 27){
                        this.pop()
                    }
                })
                var $dialog = $('.dialog-wrap');
                // 移动端改成类似toast，通过更改样式，既不需要增加toast组件，也不需要更改代码，统一pop方法
                if(screen.width < 700 && !this.getPopPara.wapGoDialog){
                    $dialog.addClass('toast-wrap');
                    setTimeout(()=>{
                        this.pop();
                        $dialog.removeClass('toast-wrap');
                    }, 2000)
                }
                //调整弹窗居中
                let width = $dialog.width();
                let height = $dialog.height();
                $dialog.css('marginTop', - height/2);
                $dialog.css('marginLeft', - width/2);
                // 弹窗建立的初始化函数
                let fn = this.getPopPara.init;
                typeof fn == 'function' && fn();
            }else{
                // 弹窗关闭时
                // 注销弹窗打开时注册的事件
                $(document).unbind('keydown')
                // 弹窗消失回调
                let fn = this.getPopPara.destroy
                typeof fn == 'function' && fn()
            }
        }
    }
}
```

#### 4.3.3 pop/toast组件参数格式化代码
为了使用方便，我们在使用的时候进行了简写。为了让组件能识别，需要在vuex的action里对传入的参数格式化。
```js
function pop({dispatch}, para) {
    // 如果没有传入任何参数，默认关闭弹窗
    if(para === undefined){
        para = {}
    }
    // 如果只传入字符串，格式化内容为content的para对象
    if(typeof para === 'string'){
        para = {
            content: para
        }
    }
    // 设置默认值
    para.pop = !para.content? false: true
    para.showClose = para.showClose === undefined? true: para.showClose
    para.title = para.title === undefined? '温馨提示': para.title
    para.wapGoDialog = !!para.wapGoDialog
    // 没有传参数
    if(!para.btn1){
        para.btn1 = '我知道了|normal'
    }
    // 没有传class
    if(para.btn1.indexOf('|') === -1){
        para.btn1 = para.btn1 + '|primary'
    }
    let array1 = para.btn1.split('|')
    para.btn1Text = array1[0]
    // 可能会传多个class
    for(let i=1,len=array1.length; i<len; i++){
        if(i==1){
            // class为disabled属性不加'btn-'
            para.btn1Css = array1[1]=='disabled'? 'disabled': 'btn-' + array1[1]
        }else{
            para.btn1Css = array1[i]=='disabled'? ' disabled': para.btn1Css + ' btn-' + array1[i]
        }
    }

    if(para.btn2){
        if(para.btn2.indexOf('|') === -1){
            para.btn2 = para.btn2 + '|normal'
        }
        let array2 = para.btn2.split('|')
        para.btn2Text = array2[0]
        for(let i=1,len=array2.length; i<len; i++){
            if(i==1){
                para.btn2Css = array2[1]=='disabled'? 'disabled': 'btn-' + array2[1]
            }else{
                para.btn2Css = array2[i]=='disabled'? ' disabled': para.btn2Css + ' btn-' + array2[i]
            }
        }
    }
    dispatch('POP', para)
}
```

为了让移动端兼容pop弹窗组件，我们采用mediaQuery对移动端样式进行了更改。增加参数`wapGoDialog`，表明我们在移动端时，要不要走弹窗，默认false，走toast。这样可以一套代码就可以兼容pc和wap。

## 后记
这里主要分析了下后台和数据库，而且比较简单，大家可以去看源码。总之，这是一个不错的前端入手后台和数据库的例子。功能比较丰富，而且可以学习下vue.js。