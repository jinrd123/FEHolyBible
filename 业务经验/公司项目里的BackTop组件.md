# 公司项目里的BackTop组件

感觉是有问题的，但多少也有点可以学习与总结的地方。

## 代码

~~~vue
<template>
    <transition name="fade">
        <div
            class="back-top"
            @click="goScrollTop"
        >
            <img src="../assets/images/backtop.png"/>
        </div>
    </transition>
</template>

<script lang="ts">
import {defineComponent} from 'vue';

export default defineComponent({
    setup() {
        const goScrollTop = () => {
            const box = document.querySelector('.wrap');
            // box && box.scrollTo({
            //     top: 0,
            //     behavior: 'smooth'
            // });
            let scrollToptimer = setInterval(function () {
                if (box) {
                    let top = box.scrollTop;
                    let speed = top / 4;
                    if (box.scrollTop !== 0) {
                        box.scrollTop -= speed;
                    } else {
                        box.scrollTop -= speed;
                    }
                    if (top === 0) {
                        clearInterval(scrollToptimer);
                    }
                }
            }, 30);
        };
        return {
            goScrollTop
        };
    }
});
</script>

<style lang="less">
.back-top {
    width: 46px;
    height: 46px;
    background: #000;
    opacity: .45;
    border-radius: 4px;
    position: fixed;
    left: calc(~'1200px + (100% - 1200px)/2');
    bottom: 100px;
    cursor: pointer;
    img {
        width: 26px;
        margin: 10px 0 0 9px;
    }
}
.fade-enter-active {
    transition: all 0.3s ease;
}

.fade-leave-active {
    transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>
~~~

## 分析与思考

样式啥的不重要，主要看结构和逻辑

说白了，结构就是一个`div`绑定一个点击事件`goScrollTop`，这个点击事件做了一件事：把滚动元素滚到顶部

### 问题一

`goScrollTop`中先去拿滚动元素：`const box = document.querySelector('.wrap');`，如果想抽象成一个更通用的BackTop，滚动元素不应该是写死的，而应该是消费组件（父组件）给我们传进来的

### 问题二

组件的作者明显想通过vue提供的<transition>组件来给`backTop`组件添加显示与隐藏动画，毕竟<transition>就是干这事的

（提到transition组件多说两句，css针对`display:none`与`display:block`这种动态切换导致的元素隐藏与消失，`transition`过渡效果是无效的，因为`display:none`的元素毕竟都没存在于页面文档流中，`display`非`none`之后才有了`transition`属性，自然`transition`过渡无法生效，所以`Vue`给我们提供了`transition`组件，当时就觉着这个组件的意义何在呢，为了简化c3的过渡与动画的语法？？现在明白它的意义了）

但是backTop组件内部并没有维护一个按钮显示与隐藏的控制变量，这个变量自然和滚动元素的`scrollTop`有关，`scrollTop`到达某个值显示，否则隐藏。但公司的项目里，对`BackTop`组件的使用方式是：

~~~html
// 某个backTop的消费组件
<transition>
    <back-top v-if="backShow"/>
</transition>
~~~

即然把显示与隐藏的控制权交给父组件了，`BackTop`组件内部的<transition>其实就完全没必要存在了，自然`BackTop`组件内的vue动画也没必要存在了（都写在父组件里了）。

### 值的学习的地方

让我写滚动到顶部的逻辑，就是上面源码里注释掉的部分，没啥好说的，但我们思考一下它这样写：

~~~typescript
let scrollToptimer = setInterval(function () {
    if (box) {
        let top = box.scrollTop;
        let speed = top / 4;
        if (box.scrollTop !== 0) {
            box.scrollTop -= speed;  // 速度控制：越靠近底部，上滚的速度越快
        } else { // 我不知道这个else存在的意义，我也没测试，单从逻辑上来讲直接删了吧。可能是scrollTop值判等不准确的原因（一般scrollTop与某个值判等都是用差的绝对值小于一的逻辑来）
            box.scrollTop -= speed;
        }
        if (top === 0) {
            clearInterval(scrollToptimer);
        }
    }
}, 30);
~~~

通过修改滚动元素`scrollTop`的值来控制滚动：`box.scrollTop -= speed;`，这样写有一个好处：滚动的速度完全是按照我们的js逻辑来的，从而延伸出去，我们也可以计算一些复杂的速度变化函数，从而完全控制回到顶部的滚动速度以及滚动时间等。



### 实践

今天周五，一个需求，弄了两三天，尝试了很多种方案都没有解决，最后换设计了...现在也没啥事，就重新封装一个通用行更高的`BackTop`吧



~~~vue
<template>
    <transition name="fade">
        <div
            v-if="shouldShow"
            class="back-top"
            @click="goScrollTop"
        >
            <img src="../assets/images/backtop.png"/>
        </div>
    </transition>
</template>

<script lang="ts">
import {defineComponent, onMounted, ref, onBeforeUnmount} from 'vue';

export default defineComponent({
    props: {
        scrollDom: {
            type: HTMLElement,
            required: true
        },
        visibleHeight: {
            type: Number,
            default: 1
        }
    },
    setup(props) {
        // 显示隐藏相关变量
        const shouldShow = ref(false);
        // 监听滚动元素的滚动行为
        onMounted(() => {
            props.scrollDom.addEventListener("scroll", handleScroll);
        });
        onBeforeUnmount(() => {
            props.scrollDom.removeEventListener("scroll", handleScroll);
        });

        // 滚动相关回调
        const throttleTag = ref(false);
        const handleScroll = () => {
            if (throttleTag.value) {
                return;
            }
            throttleTag.value = true;
            setTimeout(() => {
                const currentScrollTop = props.scrollDom.scrollTop;
                shouldShow.value = currentScrollTop >= props.visibleHeight;
                throttleTag.value = false;
            }, 30);
        };

        // 点击事件回调
        const goScrollTop = () => {
            props.scrollDom.scrollTo({
                top: 0,
                behavior: 'smooth'
            });
        };

        return {
            shouldShow,
            goScrollTop,
            throttleTag
        };
    }
});
</script>

<style lang="less">
.back-top {
    width: 46px;
    height: 46px;
    background: #000;
    opacity: .45;
    border-radius: 4px;
    position: fixed;
    left: calc(~'1200px + (100% - 1200px)/2');
    bottom: 100px;
    cursor: pointer;
    img {
        width: 26px;
        margin: 10px 0 0 9px;
    }
}
.fade-enter-active {
    transition: all 1s ease;
}

.fade-leave-active {
    transition: all 1s ease;
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
}
</style>

~~~

#### 遇到的问题

1. 逻辑上面，这个组件貌似完全封闭，在我公司的项目里也能正常运作，但并不能直接照抄，大部分业务场景中会无效，因为这里`props.scrollDom`默认为普通的某个dom元素（历史原因，公司里的项目滚动元素不是`document.body`，而是最外层的一个`div`），这会导致什么呢？其实是我一直当作bug处理的场景：

   1. 滚动元素为`body`，那么添加`scroll`监听要用`window.addEventListener`；访问`body`元素的`scrollTop`要用`document.documentElement.scrollTop`去访问。
   2. 滚动元素为普通的某个dom（`body`的子元素），那么添加`scroll`监听以及访问`scrollTop`都可以直接用滚动元素本身。

   所以，大部分业务场景中滚动元素为`body`，需要修改上面的添加监听与访问`scrollTop`的逻辑

2. 语法上面，我在`setup`中尝试解构`props`：`const { scrollDom, visibleHeight} = props;`，然后模版中直接用结构出来的变量，但是报错了，说什么解构之后元素丢失响应式。

   我仔细观察了一下（从vue的使用角度，而不是从源码角度），大概是这样的：父元素中给子组件传递`props`属性，传递的是一个`ref`响应式变量，然后子组件的`setup`中直接通过`props.xxx`去使用父元素传来的属性，应该是通过这层传递，父元素中传给子组件所有的`ref`变量组成了子组件中的`props`，然后`props`是一个`reactive`响应式对象，解构它得到的属性自然都是普通属性，报错。