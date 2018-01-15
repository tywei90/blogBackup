---
title: 前端页面制作工具
date: 2018-01-11 20:50:29
tags:    
    - 页面制作
    - pagemaker
    - react
    - redux
    - immutable
reward: true
toc: true
---
> *版权声明：本文为博主原创文章，未经博主允许不得转载！*

## 前言
pagemaker是一个前端页面制作工具，方便产品，运营和视觉的同学迅速开发简单的前端页面，从而可以解放前端同学的工作量。此项目创意来自网易乐得内部项目[nfop](http://nfop.ms.netease.com/)中的pagemaker项目。原来项目的前端是采用jquery和模板ejs做的，每次组件的更新都会重绘整个dom，性能不是很好。因为当时react特别火，加上项目本身的适合，最后决定采用react来试试水。因为原来整个项目是包含很多子项目一起，所以后台的实现也没有参考，完全重写。  

本项目只是原来项目的简单实现，去除了用的不多和复杂的组件。但麻雀虽小五脏俱全，本项目采用了react的一整套技术栈，适合那些对react有过前期学习，想通过demo来加深理解并动手实践的同学。项目的[线上地址](https://pagemaker.wty90.com/)、[github地址](https://github.com/tywei90/pagemaker_production)，欢迎大家star。建议学习本demo的之前，先学习/复习下相关的知识点：[React 技术栈系列教程](http://www.ruanyifeng.com/blog/2016/09/react-technology-stack.html)、[Immutable 详解及 React 中实践](https://zhuanlan.zhihu.com/p/20295971?columnSlug=purerender)。
<!-- more -->
## 一、功能特点
0. 组件丰富。有标题、图片、按钮、正文、音频、视频、统计、jscss输入。
0. 实时预览。每次修改都可以立马看到最新的预览。
0. 支持三种导入方式，支持导出配置文件。
0. 支持Undo/Redo操作。(组件个数发生变化为触发点)
0. 可以随时发布、修改、删除已发布的页面。
0. 每个页面都有一个发布密码，从而可以防止别人修改。
0. 页面前端架构采用react+redux，并采用immutable数据结构。可以将每次组件的更新最小化，从而达到页面性能的最优化。
0. 后台对上传的图片自动进行压缩，防止文件过大
0. 适配移动端 


## 二、用到的技术
### 1. 前端
1. React
2. Redux
3. React-Redux
4. Immutable
4. React-Router
5. fetch
6. es6
7. es7

### 2. 后台
1. Node
2. Express

### 3. 工具
1. Webpack
2. Sass
3. Pug


## 三、脚手架工具
因为项目用的技术比较多，采用脚手架工具可以省去我们搭建项目的时间。经过搜索，我发现有三个用的比较多：
1. [create-react-app](https://github.com/facebookincubator/create-react-app)   ![create-react-app star数](/assets/blogImg/git1.png "create-react-app star数")
2. [react-starter-kit](https://github.com/kriasoft/react-starter-kit#readme)   ![react-starter-kit star数](/assets/blogImg/git2.png "react-starter-kit star数")
3. [react-boilerplate](https://github.com/react-boilerplate/react-boilerplate)   ![react-boilerplate star数](/assets/blogImg/git3.png "react-boilerplate star数")  

github上的star数都很高，第一个是Facebook官方出的react demo。但是看下来，三个项目都比较庞大，引入了很多不需要的功能包。后来搜索了下，发现一个好用的脚手架工具：[yeoman](http://yeoman.io/learning/)，大家可以选择相应的generator。我选择的是[react-webpack](https://github.com/react-webpack-generators/generator-react-webpack#readme)。项目比较清爽，需要大家自己搭建redux和immutable环境，以及后台express。其实也好，锻炼下自己构建项目的能力。


## 四、工程目录分析
工程目录如下：  

![工程目录](/assets/blogImg/gc.jpg "工程目录")  

* data是用来存放数据文件的。因为数据比较简单，本项目没有采用数据库，直接用文件方式来存储。
* files是存放上传文件和下载的中间文件。
* public是最后打包生成文件的目录
* release目录是用来存放发布的静态页面目录
* server是服务的代码
* src是整个前端工程目录。action和reducer存放在各自文件夹内，index.js是入口文件。fonts文件夹存放字体文件的，采用[阿里字体库](iconfont.cn)。
* views存放前端pug模板文件的
* .babelrc文件是用来配置比如支持es6，es7等最新特性的，react, antd按需加载等。


## 五、核心代码分析
### 1. Store
Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。
```js
import { createStore } from 'redux';
import { combineReducers } from 'redux-immutable';

import unit from './reducer/unit';
// import content from './reducer/content';

let devToolsEnhancer = null;
if (process.env.NODE_ENV === 'development') {
    devToolsEnhancer = require('remote-redux-devtools');
}

const reducers = combineReducers({ unit });
let store = null;
if (devToolsEnhancer) {
    store = createStore(reducers, devToolsEnhancer.default({ realtime: true, port: config.reduxDevPort }));
}
else {
    store = createStore(reducers);
}
export default store;
```

Redux 提供createStore这个函数，用来生成 Store。由于整个应用只有一个 State 对象，包含所有数据，对于大型应用来说，这个 State 必然十分庞大，导致 Reducer 函数也十分庞大。Redux 提供了一个 combineReducers 方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。当然，我们这里只有一个 unit 的 Reducer ，拆不拆分都可以。  

devToolsEnhancer是个中间件（middleware）。用于在开发环境时使用Redux DevTools来调试redux。

### 2. Action
Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。
```js
import Store from '../store';

const dispatch = Store.dispatch;

const actions = {
    addUnit: (name) => dispatch({ type: 'AddUnit', name }),
    copyUnit: (id) => dispatch({ type: 'CopyUnit', id }),
    editUnit: (id, prop, value) => dispatch({ type: 'EditUnit', id, prop, value }),
    removeUnit: (id) => dispatch({ type: 'RemoveUnit', id }),
    clear: () => dispatch({ type: 'Clear'}),
    insert: (data, index) => dispatch({ type: 'Insert', data, index}),
    moveUnit: (fid, tid) => dispatch({ type: 'MoveUnit', fid, tid }),
};

export default actions;
```

State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。代码中，我们定义了actions对象，他有很多属性，每个属性都是函数，函数的输出是派发了一个action对象，通过Store.dispatch发出。action是一个包含了必须的type属性，还有其他附带的信息。

### 3. Immutable
Immutable Data 就是一旦创建，就不能再被更改的数据。对 Immutable 对象的任何修改或添加删除操作都会返回一个新的 Immutable 对象。详细介绍，推荐知乎上的[Immutable 详解及 React 中实践](https://zhuanlan.zhihu.com/p/20295971?columnSlug=purerender)。我们项目里用的是Facebook 工程师 Lee Byron 花费 3 年时间打造的[immutable.js库](https://github.com/facebook/immutable-js/)。具体的API大家可以去官网学习。

熟悉 React 的都知道，React 做性能优化时有一个避免重复渲染的大招，就是使用 `shouldComponentUpdate()`，但它默认返回 `true`，即始终会执行 `render()` 方法，然后做 Virtual DOM 比较，并得出是否需要做真实 DOM 更新，这里往往会带来很多无必要的渲染并成为性能瓶颈。当然我们也可以在 `shouldComponentUpdate()` 中使用使用 deepCopy 和 deepCompare 来避免无必要的 `render()`，但 deepCopy 和 deepCompare 一般都是非常耗性能的。  

Immutable 则提供了简洁高效的判断数据是否变化的方法，只需 `===`(地址比较) 和 `is`( 值比较) 比较就能知道是否需要执行 `render()`，而这个操作几乎 0 成本，所以可以极大提高性能。修改后的 `shouldComponentUpdate` 是这样的：
```js
import { is } from 'immutable';

shouldComponentUpdate: (nextProps = {}, nextState = {}) => {
  const thisProps = this.props || {}, thisState = this.state || {};

  if (Object.keys(thisProps).length !== Object.keys(nextProps).length ||
      Object.keys(thisState).length !== Object.keys(nextState).length) {
    return true;
  }

  for (const key in nextProps) {
    if (thisProps[key] !== nextProps[key] || ！is(thisProps[key], nextProps[key])) {
      return true;
    }
  }

  for (const key in nextState) {
    if (thisState[key] !== nextState[key] || ！is(thisState[key], nextState[key])) {
      return true;
    }
  }
  return false;
}
```

使用 Immutable 后，如下图，当红色节点的 state 变化后，不会再渲染树中的所有节点，而是只渲染图中绿色的部分：

![immutable演示](/assets/blogImg/immutable.jpg "immutable演示")  

本项目中，我们采用支持 class 语法的 [pure-render-decorator](http://link.zhihu.com/?target=https%3A//github.com/felixgirault/pure-render-decorator) 来实现。我们希望达到的效果是：当我们编辑组件的属性时，其他组件并不被渲染，而且preview里，只有被修改的preview组件update，而其他preview组件不渲染。为了方便观察组件是否被渲染，我们人为的给组件增加了data-id的属性，其值为`Math.random()`的随机值。效果如下图所示：  

![immutable实际效果图](/assets/blogImg/immutable.gif "immutable实际效果图")  


### 4. Reducer
Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。
```js
import immutable from 'immutable';

const unitsConfig = immutable.fromJS({
    META: {
        type: 'META',
        name: 'META信息配置',
        title: '',
        keywords: '',
        desc: ''
    },
    TITLE: {
        type: 'TITLE',
        name: '标题',
        text: '',
        url: '',
        color: '#000',
        fontSize: "middle",
        textAlign: "center",
        padding: [0, 0, 0, 0],
        margin: [10, 0, 20, 0]
    },
    IMAGE: {
        type: 'IMAGE',
        name: '图片',
        address: '',
        url: '',
        bgColor: '#fff',
        padding: [0, 0, 0, 0],
        margin: [10, 0, 20, 0]
    },
    BUTTON: {
        type: 'BUTTON',
        name: '按钮',
        address: '',
        url: '',
        txt: '',
        margin: [
            0, 30, 20, 30
        ],
        buttonStyle: "yellowStyle",
        bigRadius: true,
        style: 'default'
    },
    TEXTBODY: {
        type: 'TEXTBODY',
        name: '正文',
        text: '',
        textColor: '#333',
        bgColor: '#fff',
        fontSize: "small",
        textAlign: "center",
        padding: [0, 0, 0, 0],
        margin: [0, 30, 20, 30],
        changeLine: true,
        retract: true,
        bigLH: true,
        bigPD: true,
        noUL: true,
        borderRadius: true
    },
    AUDIO: {
        type: 'AUDIO',
        name: '音频',
        address: '',
        size: 'middle',
        position: 'topRight',
        bgColor: '#9160c3',
        loop: true,
        auto: true
    },
    VIDEO: {
        type: 'VIDEO',
        name: '视频',
        address: '',
        loop: true,
        auto: true,
        padding: [0, 0, 20, 0]
    },
    CODE: {
        type: 'CODE',
        name: 'JSCSS',
        js: '',
        css: ''
    },
    STATISTIC: {
        type: 'STATISTIC',
        name: '统计',
        id: ''
    }
})

const initialState = immutable.fromJS([
    {
        type: 'META',
        name: 'META信息配置',
        title: '',
        keywords: '',
        desc: '',
        // 非常重要的属性，表明这次state变化来自哪个组件！
        fromType: ''
    }
]);


function reducer(state = initialState, action) {
    let newState, localData, tmp
    // 初始化从localstorage取数据
    if (state === initialState) {
        localData = localStorage.getItem('config');
        !!localData && (state = immutable.fromJS(JSON.parse(localData)));
        // sessionStorage的初始化
        sessionStorage.setItem('configs', JSON.stringify([]));
        sessionStorage.setItem('index', 0);
    }
    switch (action.type) {
        case 'AddUnit': {
            tmp = state.push(unitsConfig.get(action.name));
            newState = tmp.setIn([0, 'fromType'], action.name);
            break
        }
        case 'CopyUnit': {
            tmp = state.push(state.get(action.id));
            newState = tmp.setIn([0, 'fromType'], state.getIn([action.id, 'type']));
            break
        }
        case 'EditUnit': {
            tmp = state.setIn([action.id, action.prop], action.value);
            newState = tmp.setIn([0, 'fromType'], state.getIn([action.id, 'type']));
            break
        }
        case 'RemoveUnit': {
            const type = state.getIn([action.id, 'type']);
            tmp = state.splice(action.id, 1);
            newState = tmp.setIn([0, 'fromType'], type);
            break
        }
        case 'Clear': {
            tmp = initialState;
            newState = tmp.setIn([0, 'fromType'], 'ALL');
            break
        }
        case 'Insert': {
            tmp = immutable.fromJS(action.data);
            newState = tmp.setIn([0, 'fromType'], 'ALL');
            break
        }
        case 'MoveUnit':{
            const {fid, tid} = action;
            const fitem = state.get(fid);
            if (fitem && fid != tid) {
                tmp = state.splice(fid, 1).splice(tid, 0, fitem);
            } else {
                tmp = state;
            }
            newState = tmp.setIn([0, 'fromType'], '');
            break;
        }
        default:
            newState = state;
    }
    // 更新localstorage，便于恢复现场
    localStorage.setItem('config', JSON.stringify(newState.toJS()));

    // 撤销，恢复操作(仅以组件数量变化为触发点，否则存储数据巨大，也没必要)
    let index = parseInt(sessionStorage.getItem('index'));
    let configs = JSON.parse(sessionStorage.getItem('configs'));
    if(action.type == 'Insert' && action.index){
        sessionStorage.setItem('index', index + action.index);
    }else{
        if(newState.toJS().length != state.toJS().length){
            // 组件的数量有变化，删除历史记录index指针状态之后的所有configs，将这次变化的config作为最新的记录
            configs.splice(index + 1, configs.length - index - 1, JSON.stringify(newState.toJS()));
            sessionStorage.setItem('configs', JSON.stringify(configs));
            sessionStorage.setItem('index', configs.length - 1);
        }else{
            // 组件数量没有变化，index不变。但是要更新存储的config配置
            configs.splice(index, 1, JSON.stringify(newState.toJS()));
            sessionStorage.setItem('configs', JSON.stringify(configs));
        }
    }
    
    // console.log(JSON.parse(sessionStorage.getItem('configs')));
    return newState
}

export default reducer;
```

Reducer是一个函数，它接受Action和当前State作为参数，返回一个新的State。unitsConfig是存储着各个组件初始配置的对象集合，所有新添加的组件都从里边取初始值。State有一个初始值：initialState，包含META组件，因为每个web页面必定有一个META信息，而且只有一个，所以页面左侧组件列表里不包含它。

reducer会根据action的type不同，去执行相应的操作。但是一定要注意，immutable数据操作后要记得赋值。每次结束后我们都会去修改fromType值，是因为有的组件，比如AUDIO、CODE等修改后，预览的js代码需要重新执行一次才可以生效，而其他组件我们可以不用去执行，提高性能。  

当然，我们页面也做了现场恢复功能(localStorage)，也得益于immutable数据结构，我们实现了Redo/Undo的功能。Redo/Undo的功能仅会在组件个数有变化的时候计作一次版本，否则录取的的信息太多，会对性能造成影响。当然，组件信息发生变化我们是会去更新数组的。


### 5. 工作流程
如下图所示：  
![redux流程图](/assets/blogImg/redux_flow.jpg "redux流程图")  

用户能接触到的只有view层，就是组件里的各种输入框，单选多选等。用户与之发生交互，会发出action。React-Redux提供connect方法，用于从UI组件生成容器组件。connect方法接受两个参数：mapStateToProps和mapDispatchToProps，按照React-Redux的API，我们需要将Store.dispatch(action)写在mapDispatchToProps函数里边，但是为了书写方便和直观看出这个action是哪里发出的，我们没有遵循这个API，而是直接写在在代码中。

然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。 Reducer 会返回新的 State 。State 一旦有变化，Store 就会调用监听函数。在React-Redux规则里，我们需要提供mapStateToProps函数，建立一个从（外部的）state对象到（UI组件的）props对象的映射关系。mapStateToProps会订阅 Store，每当state更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发UI组件的重新渲染。大家可以看我们content.js组件的最后代码：
```js
export default connect(
    state => ({
        unit: state.get('unit'),
    })
)(Content);
```
connect方法可以省略mapStateToProps参数，那样的话，UI组件就不会订阅Store，就是说 Store 的更新不会引起 UI 组件的更新。像header和footer组件，就是纯UI组件。

为什么我们的各个子组件都可以拿到state状态，那是因为我们在最顶层组件外面又包了一层<Provider> 组件。入口文件index.js代码如下：
```js
import "babel-polyfill";
import React from 'react';
import ReactDom from 'react-dom';
import { Provider } from 'react-redux';
import { Router, Route, IndexRoute, browserHistory } from 'react-router';

import './index.scss';

import Store from './store';

import App from './components/app';

ReactDom.render(
    <Provider store={Store}>
        <Router history={browserHistory}>
            <Route path="/" component={App}>

            </Route>
        </Router>
    </Provider>,
    document.querySelector('#app')
);
```
我们的react-router采用的是browserHistory，使用的是HTML5的History API，路由切换交给后台。


## 六、兼容性和打包优化
### 1. 兼容性 
为了让页面更好的兼容IE9+和android浏览器，因为项目使用了babel，所以采用[babel-polyfill](https://babeljs.io/docs/usage/polyfill/)和[babel-plugin-transform-runtime](https://babeljs.io/docs/plugins/transform-runtime/)插件。

### 2. Antd按需加载
Antd完整包特别大，有10M多。而我们项目里主要是采用了弹窗组件，所以我们应该采用按需加载。只需在.babelrc文件里配置一下即可，详见[官方说明](https://ant.design/docs/react/introduce-cn#%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD)。  

### 3. webpack配置externals属性
项目最后打包的main.js非常大，有接近10M多。在网上搜了很多方法，最后发现webpack配置externals属性的方法非常好。可以利用pc的多文件并行下载，降低自己服务器的压力和流量，同时可以利用cdn的缓存资源。配置如下所示：
```js
externals: {
    "jquery": "jQuery",
    "react": "React",
    "react-dom": "ReactDOM",
    'CodeMirror': 'CodeMirror',
    'immutable': 'Immutable',
    'react-router': 'ReactRouter'
}
```

externals属性告诉webpack，如下的这些资源不进行打包，从外部引入。一般都是一些公共文件，比如jquery、react等。注意，因为这些文件从外部引入，所以在`npm install`的时候，有些依赖这些公共文件的包安装会报warning，所以看到这些大家不要紧张。经过处理，main.js文件大小降到3.7M，然后nginx配置下gzip编码压缩，最终将文件大小降到872KB。因为在移动端，文件加载还是比较慢的，我又给页面加了loading效果。