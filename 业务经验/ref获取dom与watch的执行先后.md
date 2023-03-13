# 业务背景



项目中我要根据路由的不同动态修改一个dom元素的样式，思路很简单直接，通过`ref`获取dom元素，`watch`监听路由变化，路由变化时修改dom元素的样式，于是有了如下：



# 错误代码

js：

~~~typescript
// 针对底部assess-footer宽度问题进行敏捷修复
const assessFooter = ref<HTMLElement | null>(null);
watch(() => router.currentRoute.value.path, (newValue, oldValue) => {
if (newValue.startsWith('/assess/question') || newValue.startsWith('/assess/report/psyAssess')) { // 根据路由动态切换样式
        assessFooter.value && (assessFooter.value.style.minWidth = '');
    } else if (newValue.startsWith('/assess/report') || newValue.startsWith('/assess/oreport')) {
        assessFooter.value && (assessFooter.value.style.minWidth = '1200px');
    }
}, {
    immediate: true
});
~~~

html：

~~~html
<div 
    class="assess-footer"
    ref="assessFooter"
>
    <div>
        <p>TIC智能测评中心</p>
        <img :src="TICLogo" />
    </div>
</div>
~~~



js部分没必要关注细节，总而言之`watch`回调函数的第一次执行中，`assessFooter`值为`null`，所以导致样式没改成，bug没修复...



# 分析



在执行`setup`函数时，由`watch`源码可知，如果设置了`immediate: true`，那么`job`函数（也就是`watch`的回调）会当作当前宏任务的一个同步代码去执行（多说一嘴，之后由于响应式变量变化而再次触发`watch`时，其回调函数就是被`Promise.resolve().then`包装的微任务了）

然后回忆一下利用`ref`获取dom结点的步骤，html模版中给dom元素添加`ref`属性，js中声明一个同名的`ref`变量并初始化为`null`，`setup`函数`return`这个`ref`变量，`ref`就绑定好这个dom元素了。（如果`setup`不`return`是不行的）

综上，开启了`immediate`属性的`watch`回调会在`ref`绑定dom之前执行（`watch`回调执行时`setup`还没`return`），所以开启了`immediate: true`的`watch`回调里使用通过`ref.value`访问dom只会获取`null`



# 正解



（使用`nextTick`延迟对`ref`变量的访问

~~~typescript
// 针对底部assess-footer宽度问题进行敏捷修复
const assessFooter = ref<HTMLElement | null>(null);
watch(() => router.currentRoute.value.path, (newValue, oldValue) => {
    if (newValue.startsWith('/assess/question') || newValue.startsWith('/assess/report/psyAssess')) { // 根据路由动态切换样式
        nextTick(() => {
            assessFooter.value && (assessFooter.value.style.minWidth = '');
        });
    } else if (newValue.startsWith('/assess/report') || newValue.startsWith('/assess/oreport')) {
        nextTick(() => {
            assessFooter.value && (assessFooter.value.style.minWidth = '1200px');
        });
    }
}, {
    immediate: true
});
~~~

