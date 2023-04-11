# 业务需求：

父容器一行内展示若干个标签（盒子），但是标签数量不确定，可能比较多，导致放不下，这时候就需要我们手动截取一部分，然后本着尽可能多的展示的原则，最后一个标签可能需要设置其`max-width`来限制不能超出父容器。

如下示例，红色的即为要展示的最后一个标签：

![image-20230411112731417](./记录img/卡片示例.png)





# 逻辑分析：



首先需要计算需要展示几个标签，即`numOfshouldShow`，如上示例中，标签数组为四个，`numOfshouldShow`为3。这个`numOfshouldShow`即是我们截取渲染数组`slice`函数的第二个参数，同时也表示了最后一个标签的`index`（方便我们进行逻辑判断给最后一个标签元素添加样式）

然后因为我们要给最后一个标签添加一个`max-width`，所以这个属性值我们也要计算出来，即`lastLabelWidth`。



# 计算思路：



因为我们不知道标签元素渲染出来具体是多大，所以这里我们`numOfshouldShow`一开始设置为`Infinity`，即数组全部渲染，然后在`setup`中把计算逻辑放在`nextTick`中，这样做的目的就是我们在`nextTick`中可以访问模版中已经渲染上去的元素的大小等，计算完了，修改`numOfshouldShow`，这样触发`numOfshouldShow`的副作用，模版重新渲染。

由于初次渲染非常快，所以根本感知不到第一次渲染（全部标签都渲染）的效果，或者说是vue内部的渲染机制原因。

具体的计算算法，考虑的方面我就不记录了，现在需要时间去做其他事情，这个也不是难点，本次记录的重点就在于**利用初次渲染，在nextTick中拿到一些样式属性方便计算，然后修改响应式变量，触发重新渲染。**





# 代码实现：

模版部分：

~~~html
<div 
	...父容器属性
>
    <span 
        v-for="(i, index) in test.slice(0, numOfshouldShow)"  // test数组：["测试字段1", "测试3123", "dsaffdsafdsaf", "12312321"]
        :key="i"
        :style="{
            'maxWidth': ((index + 1)==numOfshouldShow && isOverFlow) ? `${lastLabelWidth}px` : '',
            'backgroundColor': ((index + 1)==numOfshouldShow && isOverFlow) ? 'red' : '',
        }"
    >
        {{i}}
    </span>
</div>
~~~

js部分：

~~~typescript
// 关于最后一个label大小的计算
const test = ["测试字段1", "测试3123", "dsaffdsafdsaf", "12312321"];
const labelBox = ref(null); // 获取容器
let numOfshouldShow = ref(Infinity); // 最后要显示的子元素数量
const containerWidth = 250; // label容器的大小
let marginRight = 10; // 标签盒子的右边距
const lastLabelWidth = ref(0); // 计算目标：最后一个元素的max-width值
const isOverFlow = ref(false); // 标识是否有标签溢出
nextTick(() => {
    if (labelBox.value) {
        let fisrtRenderedChildrenCount = (labelBox.value as unknown as Element).childElementCount;
        let leftWidth = containerWidth;
        let num = 0;
        for (let i = 0; i < fisrtRenderedChildrenCount; i++) {
            if (leftWidth - ((labelBox.value as unknown as Element).children[i].clientWidth + marginRight) 
            >= 0) {
                num++;
                leftWidth -= (labelBox.value as unknown as Element).children[i].clientWidth + marginRight;
            } else {
                num++;
                isOverFlow.value = true;
                break;
            }
        }
        numOfshouldShow.value = num;
        lastLabelWidth.value = leftWidth;
    }
});
~~~

