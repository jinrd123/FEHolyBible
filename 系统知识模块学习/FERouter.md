# 前言

下面我们手写实现一下前端路由（玩具版），**但是不要小看丐版，相较于成熟的开源源码库，丐版能让我更清晰的抓住事物的核心**，我会尽可能的按照我编码时的逻辑复现出来最终的代码，并且我会把我学习过程中遇到的前置知识也写在文章中，可以节省大家查资料的时间，话不多说，直接开搞。

在动手之前，我觉着有必要再清晰一下我们的目标，我们要实现前端路由，啥是前端路由，说白了就是浏览器打开一个页面之后，浏览器地址栏发生变化，然后页面上的内容也随之呈现与地址栏对应的内容，但是这个过程中浏览器并没有刷新。

路由配置一般就是`path(url)`与`component(要展示的内容)`之间的映射，常见的路由配置如：

~~~js
const routes = [
  {
    path: '/',
    component: xxx
  },
  {
    path: '/center',
    component: xxx
  },
  {
    path: '/about',
    component: xxx
  },
]
~~~

好了，目标明确，开搞。



# hash模式

兄弟们，咱先把hash模式秒了开开胃。

首先整理一下完整的前端路由改变的一个逻辑流程：

1. 首先我们需要有方法让浏览器地址栏发生改变——在保证页面不刷新的前提下
2. 浏览器地址栏发生改变了，我们需要能感知到这个改变，也就是我们最熟悉的事件监听，即对地址栏改变这个事件进行监听
3. 在监听的回调函数中，我们修改页面的内容，即展示新地址栏对应的组件

下面我们就按照这三个步骤来实现对hash路由的模拟



## 改变浏览器地址栏url

我们知道页面的[url组成(相关文章)](https://blog.csdn.net/weixin_42752574/article/details/102852861)最后一部分是锚点（hash）部分，也就是`#xxx`的内容，这一部分的特点就是http请求并不携带这部分的内容，并且修改hash并不会导致浏览器的刷新（可以随意打开一个网址，然后在最后面写一个`#`开头的字符串比如`#dasfhuasdhf`，页面并没有刷新），除了这种手动修改的方式修改hash的内容，还可以通过`a`标签的`href`属性来提供给用户修改hash的交互接口，如下`html`结构，通过三个`a`标签提供给了用户修改hash的三个按钮，可以类比`vue`中的`<router-link>`，`react`中的`<Link>`：

~~~html
<div class="router-nav">
  <a href="/#/" class="nav-item">首页</a>
  <a href="/#/center" class="nav-item">个人中心</a>
  <a href="/#/about" class="nav-item">关于</a>
</div>
<div class="page-content"></div> // 上面是路由导航按钮，page-content里面我们就模拟路由组件的展示，类比router-view
~~~



## 对浏览器地址hash部分改变的监听

说个后话，为啥hash模式相较于history实现起来要简单一些（虽然写完发现都不难😄），就是因为当hash改变时会触发`window`对象的`hashchange`事件，相当于浏览器原生环境帮咱铺平道路了，泰裤辣，直接写代码吧，思路在注释里：

~~~js
window.addEventListener("hashchange", (hashchangeEvent) => {
  // 1. 总逻辑——修改div.page-content里面的内容
  // 2. hashchangeEvent对象里面有个newURL属性，就是在hashchange之后浏览器地址栏里的完整内容，稍微一处理我们就可以拿到#后面的内容(#后面的内容方便与路由配置里的path进行匹配)
  // 3. 新的hash字符串即为event.newURL.split("#")[1]
  // 4. 上面拿到当前的hash字符串然后去对比路由配置，展示需要展示的组件即可，可以写一个matchComponent函数实现
});
~~~



## 实现`matchComponent`函数，实现hash与页面展示内容的匹配

~~~js
const matchComponent = (routes, path) => { // routes即为路由配置数组，path即为浏览器地址栏中当前的hash字符串
  const { component } = routes.find(route => route.path === path); // 寻找routes数组中与传进来的path匹配的路由配置对象
  // 将component的内容呈现在div.page-content里
}
~~~



## 最终代码

上面是用面向过程的方式实现的每个步骤，下面用面向对象的方式串联一下上面的整个思路。（面向对象，说白了就是将路由相关的逻辑放到一个类里，然后初始化一下类的实例就相当于把构造函数里的逻辑都执行了，只是图一个更规范的代码结构罢了）

为了整体给一个逻辑全貌，我下面就用注释的形式进行代码逻辑的辅助解释

代码实现（阅读）顺序：`constructor函数 ——> refresh函数 ——> matchComponent函数`

~~~js
class HashRouter {
  
  
  // constructor的目标就是给hashchange事件添加监听，回调函数refresh的逻辑就是更改页面需要展示的内容
  constructor(routes = []) {
    this.routes = routes; // 路由映射数组
    this.currentHash = ""; // 记录当前地址栏中的hash字符串，也就对应了应该展示的路由组件
    // 这里需要永久改变一下refresh的指向，改为HashRouter实例，因为它作为事件回调传递给addEventListener之后的调用就跟HashRouter实例没关系了，但是refresh方法里又要用到其他类方法，如执行this.matchComponent()，所以需要让this始终指向HashRouter实例，才能正确访问类方法与类属性
    this.refresh = this.refresh.bind(this);
    // 因为我们的页面一上来加载并没有触发hashchange事件，为了让一开始路由组件也正常显示出来一个，这里我们也监听一下load事件让首屏不空白（本玩具的边缘情况😄）
    window.addEventListener("load", this.refresh);
    // 监听hashchange事件，这才是核心，---------- 下面我们看this.refresh方法 ----------
    window.addEventListener("hashchange", this.refresh);
  }
  
  
  // refresh方法作为hashchange和load事件的回调函数，自然可以接收到一个事件对象event，但是逻辑里需要区别一下load与hashchange不同的事件对象的处理
  refresh(event) {
    if (event.newURL) { // 对于hashchange事件的event存在newURL属性，即为整个浏览器地址栏里的url字符串，截取一下#后面的部分记录到this.currentHash属性上
      this.currentHash = event.newURL.split("#")[1];
    } else { // 对于load事件的event，可以通过window.location.hash访问到当前url里#(包括#)往后的部分，截取一下记录到this.currentHash属性上
      this.currentHash = window.location.hash.slice(1);
    }
    // 有了最新的this.currentHash属性，就可以执行matchComponent方法进行页面内容的展示了 ---------- 下面看matchComponent方法 ----------
    this.matchComponent();
  }
  
  
  matchComponent() {
    // 这里逻辑就很简单了，主打一个丐版为了突出核心，路由配置数组中component就是一个html字符串，我们拿到之后通过innerHTML修改一下dom内容就ok了
    const { component } = this.routes.find(route => route.path === this.currentHash);
    document.querySelector(".page-content").innerHTML = component;
  }
}


// 传入一个路由配置数组实例化HashRouter，即执行constructor函数
new HashRouter([
  {
    path: '/',
    component: '<div>首页</div>'
  },
  {
    path: '/center',
    component: '<div>个人中心</div>'
  },
  {
    path: '/about',
    component: '<div>关于</div>'
  },
]);
~~~

html部分——`index.html`（不用关心样式啥的，核心的html就是那几个a标签，然后还有通过`<script>`引入了一下上面我们写的js代码）：

~~~html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <style>
      body {
        margin: 0;
        padding: 0;
      }
      .container {
        width: 400px;
        height: 400px;
        border: 1px solid #000;
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
      }
      .router-nav {
        display: flex;
        justify-content: space-between;
        align-items: center;
      }
      .nav-item {
        width: 33.3%;
        border: 1px solid #000;
        text-align: center;
      }
    </style>
    <div class="container">
      <div class="router-nav">
        <a href="/#/" class="nav-item">首页</a>
        <a href="/#/center" class="nav-item">个人中心</a>
        <a href="/#/about" class="nav-item">关于</a>
      </div>
      <div class="page-content"></div>
    </div>
    <script src="./hash.js"></script>
  </body>
</html>
~~~



# history模式

先简单提一句，所谓history模式，相较于hash模式的区别就是url里没有`#`，看起来更美观一些罢了，实现起来麻烦一些，但总的实现思路与hash一样，分三步：

1. 确定url改变的方法
2. 对url改变的监听
3. 监听的回调函数中对视图的修改（感觉这一步总结为核心有点鸡肋，其实真正的核心感觉就是url改变与对url的监听）

这个模式实现起来比较麻烦，其实把握住上面总的思路，也就只是麻烦一点罢了，完全没啥难度，话不多说，开搞家人们。

（前置知识我也写了一下，大家“按需引入”即可）





## 前置知识

### 浏览器访问历史栈

说白了浏览器对与访问记录的记录用的数据结构是个栈，然后有个指针指向这个栈的某个数据，对应当前显示的网页，举个例子，我们打开浏览器，此时栈中只有当前页面，指针也指向这个记录，左上角的前进后退按钮都无效（因为只有一个记录数据），浏览器栏里输入一个新网址然后访问，新网址就被push进栈了，指针也指向了新记录，然后我们发现可以点击后退按钮了，相当于指针从栈顶移动到栈底，也就是第一个页面。

有个细节：从上面我们知道，有个所谓的“指针”，这个指针可以通过浏览器左上角的前进后退按钮在栈中移动。假设现在我们的指针不在栈顶，即可以点击浏览器的前进按钮，然后这时候我们不管是点击页面上的链接或者是从浏览器地址栏里输入一个新的网址，栈中增加的记录是基于当前指针为栈顶的子栈为基础的（不懂的话读起来就比较绕了，直接看后面例子）。举个具体的例子：打开chrome浏览器，转跳百度，点击后退（此时退回chrome了），此时浏览器地址栏输入gitee（基于chrome为栈顶的这个子栈，push进去gitee），发现此时浏览器栈还是两个记录，一个chrome，一个gitee，因为那个指针没到百度，所以百度被噶掉了。

这里我就不废话了，感兴趣的话了解一下就好了。



### window.history对象

这个是h5提供的一个对象，它上面有很多属性和方法，都是与浏览器栈相关的，这里我们介绍一下会用到的属性与api。

[阮一峰老师的讲解](https://javascript.ruanyifeng.com/bom/history.html)



#### window.history.pushState(state, title, url)

总结一下：

* `pushState`方法可以在浏览器栈中push一个新记录，但是当前**页面不会刷新**。
* 第一个参数`state`是一个自定义的对象，可以随意存放一些信息，通过`window.history.state`可以访问当前浏览器栈指针指向的记录的`state`对象（`pushState`的第一个参数），也就是当前页面对应的`state`对象。
* 第二个参数，会被浏览器忽略，所以直接传`""`
* 第三个参数，即为新的url路径字符串（基于当前路径，第三个参数作为相对路径进行拼接，随便试一下看下啥表现就知道了，不是啥重点）

注：必须要`window.history`对象去掉用`pushState`才可以正常运行（要保证`pushState`的`this`为`window.history`）



#### window.history.replaceState(state, title, url)

用法和`pushState`一样，唯一的区别是`pushState`是在栈里push，replaceState相当于先pop，再push（replace）

两者的核心特点就是，只是修改浏览器栈的记录，但是**页面并不刷新**，这正是我们前端路由要的效果



#### popstate事件

我觉着还是看一下[阮一峰老师的讲解](https://javascript.ruanyifeng.com/bom/history.html)好一些，我这里简略总结一下我们需要了解的一些特点：

* 点击浏览器的前进后退按钮会触发此事件
* 但是pushState和replaceState方法的调用不会触发此事件



## 总思路分析

第一步——url改变的方法我们已经有了，就是利用`replaceState`和`pushState`两个api，比如我们给几个按钮添加点击回调，回调逻辑中调用这两个方法即可完成浏览器地址栏url的修改；

然后就是第二件事，我们需要监听到`replaceState`和`pushState`两个方法的调用；

最后一步，和hash模式无异，路由匹配，然后修改一下页面显示的内容即可；



## 改变浏览器地址栏url

html部分，提供三个按钮：

~~~html
<div class="container">
  <div class="router-nav">
    <button class="nav-item home">首页</button>
    <button class="nav-item center">个人中心</button>
    <button class="nav-item about">关于</button>
  </div>
  <div class="page-content"></div>
</div>
~~~

js部分，给三个按钮增加交互逻辑，使之可以修改url：

~~~js
const homeButt = document.querySelector(".home");
const centerButt = document.querySelector(".center");
const aboutButt = document.querySelector(".about");

// 三个按钮都绑定了pushState行为，其实replaceState完全一样的
homeButt.addEventListener("click", () => {
    history.pushState({path: "/"}, '', './');
})
centerButt.addEventListener("click", () => {
    history.pushState({path: "/center"}, '', './center');
})
aboutButt.addEventListener("click", () => {
    history.pushState({path: "/about"}, '', './about');
})
~~~



## 对浏览器地址栏改变的监听

这里我们浏览器地址栏url的改变本质上是`pushState`方法的调用，所以对浏览器地址栏的监听问题等价转换为对`pushState`方法调用的监听——重头戏：**重写pushState方法**



### 重写pushState方法

我们的目标是重写`window.history.pushState`（以及`replaceState`），让他们在调用时可以触发某个事件，然后我们就可以对其进行监听了

~~~js
// _wrap方法对原生pushState和replaceState进行处理
let _wrap = (type) => {
  	// 先拿到pushState（replaceState）原生函数体
    let originFun = history[type];
  	// _wrap返回的这个函数要做到两件事：1. 保证pushState（replaceState）的正常功能，说白了也就是保证调用原本的pushState方法时this、arguments参数正确，而且新返回的函数返回结果也与以前的pushState一样
  	// 2. 触发一个自定义事件，使新的pushState可以被监听
    return function () {
        let result = originFun.apply(this, arguments); // 这一行代码其实就做到上面要求的第一件事了，第一，我们用apply方法保证了this的正确性，即以后再调用history.pushState(xxx)时让history对象作为this去执行pushState原生函数体，并且也正确传递了参数，然后还保留了执行结果result，最后retuen返回这个result即可。
      	// 下面三行代码，创建一个pushState事件，然后dispatch出去，就做到了让pushState方法也可被监听，把arguments挂在事件对象e上，目的就一个，pushState的第一个参数不是可以记录一些信息么，看上面给按钮绑定的回调，我们记录了一个path，我们以后就拿着这个path去对比，以得到该展示的路由组件
        let e = new Event(type);
        e.arguments = arguments;
        window.dispatchEvent(e);
        return result;
    }
}

history.pushState = _wrap("pushState");
history.replaceState = _wrap("replaceState");
~~~



### 监听pushState

（replaceState也完全一样）

~~~js
window.addEventListener("pushState", (e) => {
    // 通过matchComponent匹配路由然后页面展示即可
})
~~~



## matchComponent函数实现——>路由匹配 & 更新视图

~~~js
const matchComponent = (path) => {
  	// 跟hash的实现完全一样啊，都只是简单模拟，路由匹配与试图更新主打一个点到为止
    const {component} = routes.find(((route) => route.path === path));
    document.querySelector(".page-content").innerHTML = component ? component : routes[0].component;
}
~~~



## 收尾工作

考虑到一个问题，我们通过监听用户触发的`pushState`和`replaceState`事件进行了路由的更新展示，但是如果用户点击前进后退按钮也会改变url，但是我们并没有进行相应的逻辑处理（路由匹配和更新视图），所以给`popstate`事件也添加上回调，还有路由首屏展示问题，所以给`load`事件也添加一个回调：

~~~js
window.addEventListener("load", () => {
  	// 直接用"/"去匹配路由
    matchComponent("/");
})

window.addEventListener("popstate", (e) => {
  	// 逻辑是比较粗糙的，如果没有state，就直接默认用"/"路径去匹配了
    if(window.history.state){
        matchComponent(window.history.state.path);
    }else {
        matchComponent("/");
    }    
})
~~~



## 最终代码

上面已经对每个部分做了详细分析，下面看一下代码全貌，我做一下总结性注释：

~~~js
// 重写pushState方法
let _wrap = (type) => {
    let originFun = history[type];
    return function () {
        let result = originFun.apply(this, arguments);
        let e = new Event(type);
        e.arguments = arguments;
        window.dispatchEvent(e);
        return result;
    }
}

history.pushState = _wrap("pushState");
// history.replaceState = _wrap("replaceState");

// 给三个按钮绑定点击回调，从而可以修改url
const homeButt = document.querySelector(".home");
const centerButt = document.querySelector(".center");
const aboutButt = document.querySelector(".about");

homeButt.addEventListener("click", () => {
    history.pushState({path: "/"}, '', './');
})
centerButt.addEventListener("click", () => {
    history.pushState({path: "/center"}, '', './center');
})
aboutButt.addEventListener("click", () => {
    history.pushState({path: "/about"}, '', './about');
})

// 路由配置数组
const routes = [
    {
        path: '/',
        component: '<div>首页</div>'
    },
    {
        path: '/center',
        component: '<div>个人中心</div>'
    },
    {
        path: '/about',
        component: '<div>关于</div>'
    },
];

// 路由匹配函数
const matchComponent = (path) => {
    const {component} = routes.find(((route) => route.path === path));
    document.querySelector(".page-content").innerHTML = component ? component : routes[0].component;
}

// 对pushState事件的监听，即点击按钮调用我们改造后的pushState方法，触发pushState事件
window.addEventListener("pushState", (e) => {
    matchComponent(e.arguments[0].path);
})

// 收尾工作，处理首屏展示 & 点击浏览器左上角的前进后退按钮问题
window.addEventListener("load", () => {
    matchComponent("/");
})

window.addEventListener("popstate", (e) => {
    if(window.history.state){
        matchComponent(window.history.state.path);
    }else {
        matchComponent("/");
    }    
})
~~~



# 总结

通过上面我们的手写，其实就能很清晰的认识到一个问题，**不管hash模式还是history模式，说白了就是一句话，监听url的改变修改视图，问题再核心一点就是监听url的改变**。

然后所谓的history模式页面刷新的404问题，也非常容易理解了，本质上就是因为`pushState`方法修改了浏览器栈的记录，但是这个修改就是非常纯粹的修改——不刷新页面，不发请求。这就导致我们经过`pushState`之后的`url`实际上可能已经不是一个真实存在的网络资源了（后端根本不存在这个页面），这时候我们手动刷新页面，就会导致发送网络请求去访问这个`url`，但是不存在，所以报错404。

虽然丐版，但是前端路由的原理真的悟了。

如果对朋友们有帮助，点个赞吧鼓励一下小弟吧！



# 参考文章

[源代码](https://github.com/jinrd123/FE-router)

[面试官为啥总是喜欢问前端路由实现方式？](https://juejin.cn/post/7127143415879303204)