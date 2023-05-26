# vue项目中问题复现

js：

~~~js
const imgUrl = "@/assets/images/xxx.png";
~~~

html：

~~~vue
<img :src="imgUrl" />
~~~

vue中通过`:src`加变量的形式是无法正确访问图片资源的，因为编译时`v-bind`指定`src`相当于直接把变量值赋值给`src`，这就会导致webpack打包后资源路径的相对关系不正确，换句话说，如果我们不用`v-bind`修饰`src`，比如`src="../../xxx.jpg"`，webpack打包时内部的逻辑会保证输出的资源之间正确的相对位置关系，但是我们用`v-bind`修饰`src`，webpack打包编译模版时就会单纯的将变量值赋值给`src`，但是我们打包输出的资源相对路径关系已经改变了（相对于编译之前的项目源代码），`src`的值是项目源代码中的资源相对路径，打包后资源相对路径已经改变，理所当然报错。



# 解决方案

通过`require`动态指定资源路径：

~~~vue
<img :src="require(`../../../assets/images/assessCover/${currentRoute}.png`)">
~~~

上面问题复现中解释了那样写为啥不对，这里这样写为啥对我是不清楚的，这就涉及webpack的编译流程了，我就不妄加猜测了。对于业务中知道这样去解决就可以了。

感觉还是想胡乱（根据对webpack的理解）猜测一波，有可能通过`require`指定资源后，webpack就会把`require`当作一个模块（文件），webpack建立资源的依赖关系不就是保证各个模块之间的依赖关系。src指定为模块，webpack处理时自然就能保证资源相对位置之间的正确性。



# 坑

将路径字符串提取为一个变量，然后`:src=require(变量)`这样也是会报错行不通的。

~~~vue
const imgUrl = `../../../assets/images/assessCover/${currentRoute}.png`;

<img :src="require(imgUrl)">
~~~

肯定还是跟vue代码的模版编译相关，具体细节就不知道了。