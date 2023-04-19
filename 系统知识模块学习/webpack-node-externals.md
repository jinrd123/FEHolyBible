`webpack-node-externals`是一个`webpack`生态里的“包”，之所以说是一个“包”，因为我也不知道它属于什么定位，反正它不是`loader`，不是`plugin`，而是给`externals`配置项使用的。所以下面先来了解下`webpack`配置对象中的`externals`属性是干啥用的。



# externals

## 官方定义

**防止**将某些 `import` 的包(package)**打包**到 bundle 中，而是在运行时(runtime)再去从外部获取这些*扩展依赖(external dependencies)*。



## 官方举例

从 CDN 引入 [jQuery](https://jquery.com/)，而不是把它打包：

**index.html**

```html
<script
  src="https://code.jquery.com/jquery-3.1.0.js"
  integrity="sha256-slogkvB1K3VOkzAI8QITxV3VzpOnkeNVsKvtkYLMjfk="
  crossorigin="anonymous"
></script>
```

**webpack.config.js**

```javascript
module.exports = {
  //...
  externals: {
    jquery: 'jQuery',
  },
};
```

这样就剥离了那些不需要改动的依赖模块，换句话，下面展示的代码还可以正常运行：

```javascript
import $ from 'jquery';

$('.my-element').animate(/* ... */);
```

上面 `webpack.config.js` 中 `externals` 下指定的属性名称 `jquery` 表示 `import $ from 'jquery'` 中的模块 `jquery` 应该从打包产物中排除。 为了替换这个模块，`jQuery` 值将用于检索全局 `jQuery` 变量，因为默认的外部库类型是 `var`。



## 总结

上面是官方文档的原话，我再用我自己的话来解释一下，首先我们项目中使用了外部依赖`jQuery`，但是呢，我们在`<html>`中通过`<script>`的方式引入了`jquery`库的逻辑，所以就不想让`webpack`打包时将`jquery`库的代码逻辑打包进打包产物中了，所以如上`webpack.config.js`中我们就配置`externals`，`{jquery: 'jQuery'}`的意思就是打包时忽略对`jquery(external对象的key值)`库的逻辑引入（不要把`jquery`模块的逻辑打入产物中），但是终归我们的业务代码要使用`jquery`模块提供的功能，这时候就全局搜索`jQuery(external对象的value值)`对象代替从原来的`jquery`模块中引入的`$`。

所以，`externals`配置项的合理配置带来的直接影响就是打包产物体积的缩减。



# webpack-node-externals

有了上面对`externals`配置项的理解再来认识`webpack-node-externals`，就小kiss了

**使用方式：**

~~~js
// webpack.config.js
const nodeExternals = require('webpack-node-externals');

module.exports = {
  // ...
  target: 'node', // 表示我们预期打包的产物是在node环境下运行
  externals: [nodeExternals()], // 使用'webpack-node-externals'提供的功能：打包过程中排除一些模块打入产物
  // ...
};
~~~

**效果：**

`webpack-node-externals`（应该算是一种插件）主要用于排除` Node.js `中的内置模块和 `node_modules` 中的第三方模块，使它们不被打包进最终的输出文件中。

比如内置模块`fs`、`path`等，我们没必要把他们的代码实现也打包进我们的产物中（因为代码在`node`环境中运行，自然可以访问这些模块），然后还有我们的业务代码中所加载的`node_modules`里的依赖，也不会打包进产物中，就举个简单的例子，我们的业务代码依赖了`Lodash`库，然后还依赖了另外一个`a`库，但是`a`库也依赖了`Lodash`，这样如果不进行任何处理，打包时“引入即打入”，就会导致打包`a`库时打进去了`Lodash`，然后打包我们的业务代码时除了打入`a`库的内容(此时`a`库的经过打包处理已经包含了`Lodash`的逻辑)，因为本身对`Lodash`的引入又重复打入了一遍`Lodash`。其实合理的场景是我们整个项目的打包产物中只存在一份`Lodash`的实现，然后所有用到的地方共享即可，这也就是`webpack-node-externals`插件所做的事情。

最终效果就是减小了打包后文件的体积，提高了打包速度。