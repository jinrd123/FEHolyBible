# 业务场景

页面中耗时的异步操作（请求后端数据）之前，模版没有渲染，这不仅会导致空白屏，如果页面底部有`footer`组件的话，用户还会看到底部的`footer`与顶部的`header`相邻，影响体验。

所以有必要增加一个`loading`组件，在数据到来、数据渲染之前撑开应有的部分并提示用户正在加载：



![image-20230213202458982](./记录img/loading效果图.png)

# 效果实现

先在`html`文件中实现这个加载的动画效果：

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Loading组件</title>
</head>
<body>
    <style>
        .container {
            width: 100%;
            min-height: 100px;
            position: relative;
        }
        .container .loading-effect {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translateX(-50%);
            transform: translateY(-50%);
            width: 50px;
            height: 50px;
        }
        .container .loading-effect span {
            display: block;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #4C84FF;
            position: absolute;
            animation: load 1.0s ease infinite;
        }
        .container .loading-effect span:nth-child(1) {
            top: 0;
            left: 50%;
            margin: 0 0 0 -5px;
            animation-delay: -1s;
        }
        .container .loading-effect span:nth-child(2) {
            top: 20%;
            left: 80%;
            margin: -5px 0 0 -5px;
            animation-delay: -0.875s;
        }
        .container .loading-effect span:nth-child(3) {
            right: 0;
            top: 50%;
            margin: -5px 0 0 0;
            animation-delay: -0.75s;
        }
        .container .loading-effect span:nth-child(4) {
            top: 80%;
            left: 80%;
            margin: -5px 0 0 -5px;
            animation-delay: -0.625s;
        }
        .container .loading-effect span:nth-child(5) {
            bottom: 0;
            left: 50%;
            margin: 0 0 0 -5px;
            animation-delay: -0.5s;
        }
        .container .loading-effect span:nth-child(6) {
            top: 80%;
            left: 20%;
            margin: -5px 0 0 -5px;
            animation-delay: -0.375s;
        }
        .container .loading-effect span:nth-child(7) {
            left: 0;
            top: 50%;
            margin: -5px 0 0 0;
            animation-delay: -0.25s;
        }
        .container .loading-effect span:nth-child(8) {
            top: 20%;
            left: 20%;
            margin: -5px 0 0 -5px;
            animation-delay: -0.125s;
        }
        p {
            width: 110px;
            font-size: 14px;
            position: absolute;
            left: -13px;
            top: 42px;
        }
        @keyframes load {
            0% {
                transform: scale(1.2);
                opacity: 1;
            }
            100% {
                transform: scale(.3);
                opacity: .5;
            }
        }
    </style>
    <div class="container">
        <div class="loading-effect">
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <span></span>
            <p>正在加载中...</p>
        </div>
    </div>
</body>
</html>
~~~

动画分析：

* 说白了，不要把动画当成一个整体来看，想着乱七八糟的移动啥的。单看一个小圆点，就是一个从大到小，然后颜色（透明度）略微也有点变化的过程无限循环，动画就很好写了。
* 关于小圆点的定位，就是在一个正方形盒子里进行绝对定位，我把第一个span定位到了`0deg`的位置，然后剩下的顺时针定位，`0deg 90deg 180deg 270deg`位置好确定，然后对于右上等角上的span稍微向外偏移一点（比如右上第二个span在top25%、left75%的基础上top少5%，left多5%）loading的轨迹就是圆形了。
* 负`animation-delay`：比如`animation-delay: -1s`表示动画从运行`1s`后的样子开始呈现，然后我们规定动画的持续时间为`1s`，然后八个小圆圈只需要设置第一个`animation-delay`为`-1s`然后依次递增`-0.125s`就ok了。

小坑：

* 给小圆点(`span`)定位的时候使用负外边距来进行最后移动，不要用`transfrom: translate`，不然会和下面的动画里的属性冲突。

# 组件封装

只需要一个`loadHeight`接口属性就可以了，表示要撑开的内容的高度：

~~~typescript
<script lang="ts">
import {defineComponent} from 'vue';

export default defineComponent({
    name: 'Loading',
    props: {
        loadHeight: {
            type: String,
            requied: false,
            default: '100%'
        }
    }
});
</script>
~~~

使用时搭配条件渲染，例如vue中：

~~~vue
<template>
    <loading v-if="loading"/>
    <div
        v-else
        ref="box"
        class="wrap"
        @mousemove="getTime"
    >
        <headbar />
        <router-view/>
        <footbar />
    </div>
</template>
~~~



