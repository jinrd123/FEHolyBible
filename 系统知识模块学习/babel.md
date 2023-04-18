babel就天天低头不见抬头见，但是作为一个构建的工具，似乎从来没有真正了解过他，今天自己写项目，自己配构建文件，终于要面对我不认识babel这个事实了，狠了狠心，花了一晚上有了下面的babel入门，现在我可以自信的说”babel真真正正是我的个熟人“了。



# babel概念

Babel源码库对其的描述：`Babel is a compiler for writing next generation JavaScript.`，意思就是`babel`是一个编译器，用于编写下一代（较新版本？）的`js`代码。

[babel官方文档](https://www.babeljs.cn/docs/usage)的使用指南中的一句话我觉着非常具体的说明了`babel`的用途：`本指南将向你展示如何将 ES2015+ 语法的 JavaScript 代码编译为能在当前浏览器上工作的代码。这将涉及到新语法的转换和缺失特性的修补。`。我用我的话来简单解释一下：因为浏览器可以识别的`js`代码版本（规范）比较滞后，所以我们书写的比较新版本的（`ES6+`）`js`代码如果要在浏览器上运行，就需要`babel`将代码进行转换，这个转换做的事情大致分为两类：

* 语法降级，比如箭头函数转普通函数，将高级语法降级为一些低级的、浏览器能识别的代码
* “特性修补”，因为浏览器所支持的`js`比较滞后，一些新方法、新内置类都是不存在的，例如一些原型上或者构造函数上的方法，比如`Array.from`等等函数，还比如`Promise`、`WeakMap`等等类，所以通过`babel`处理来增加（用代码去实现、模拟）这些“特性”。



# babel常用模块

官方：”你所需要的所有的 Babel 模块都是作为独立的 npm 包发布的，并且（从版本 7 开始）都是以 `@babel` 作为冠名的。这种模块化的设计能够让每种工具都针对特定使用情况进行设计。“也就是说，有各种各样的`@bebel/xxx`库，我们大概了解几个

我们不去细究`babel`的配置语法、如何使用等细节，而是从更宏观的角度了解一下模块用了什么（**陷入知识的细枝末节==g**）



## `@babel/core`

`Babel`的核心功能模块，核心功能是啥，不就是代码转换么。

`npm init -y`新建个项目，执行`npm install --save-dev @babel/core`后我们写个小demo实测一下

~~~js
// babel-core.js
const babel = require("@babel/core");

const ans = babel.transformSync("() => { console.log('测试代码'); }");

console.log(ans.code);
~~~

然后我们`node babel-core.js`运行一下，控制台原封不动得打印出了我们传给转换函数`transfromSync`的代码段，这里起码可以说明一件事，我们输入一个代码段，最终我们拿到了返回的代码段，这么一个输入代码，反馈（输出）代码的逻辑已经很清晰了。

至于代码为什么没有变化，那就是需要借助另一个比较重要的`Babel`模块了，也就是我们的`插件`与`presets(预设)`



## 插件和预设（preset）& `@babel/cli`



### 插件（如`@babel/plugin-transform-arrow-functions`） & `@babel/cli`

有一个插件`@babel/plugin-transform-arrow-functions`，它的作用是如果`Babel`在编译代码的过程中使用了它，就会把箭头函数转化为`ES5`兼容的函数表达式。

执行`npm install --save-dev @babel/plugin-transform-arrow-functions`安装一下这个插件，在js代码中不方便使用这个插件，所以我们用命令行调用的方式使用`babel`来进行代码编译，为了能进行命令行调用我们需要安装**`@babel/cli`**，我们先来看一下`@babel/cli`安装前后的变化：安装之后，`node_modules`下出现了`./bin`文件夹，里面有一个`babel`可执行程序。

万事具备，回归正题——使用`@babel/plugin-transform-arrow-functions`插件：

命令行执行`./node_modules/.bin/babel babel-core.js --out-dir ./output_dir --plugins=@babel/plugin-transform-arrow-functions`，意思就是用`babel`程序处理`babel-core.js`文件的内容，`--out-dir`指定执行编译后代码输出位置为`./out-dir`，`--plugins=`指定处理过程中使用插件`@babel/plugin-transform-arrow-functions`。

~~~js
// babel-core.js
() => {
  console.log("babel处理之前我是一个箭头函数");
};

// ./output-dir/babel-core.js —— babel处理后输出的文件
(function () {
  console.log("babel处理之前我是一个箭头函数");
});
~~~



这样通过上面的demo，我们大概了解了`@babel/cli`模块就是提供了一个`babel`可执行程序，它存放在`./node_modules/.bin`目录下，可以在命令行使用，然后所谓的`babel插件`，从效果来说就类似一些对`babel`处理后所输出的文件的期望，换言之，`babel`编译处理过程中，使用了某种插件，编译过程就会执行这个插件的一些逻辑，从而影响输出的代码。



### preset预设（如`@babel/preset-env`）



说白了就是一个插件的集合，即一组预先设定的插件，而且`preset`支持传参，可以在预设的基础上进行更精细的编译控制。

比如`@babel/preset-env`，如果不进行任何配置，上述 preset 所包含的插件将支持所有最新的 JavaScript （ES2015、ES2016 等）特性。

安装并使用：

~~~bash
npm install --save-dev @babel/preset-env

./node_modules/.bin/babel <inputFile> --out-dir <outputDir> --presets=@babel/env #编译过程中使用@babel/env预设
~~~



## `@babel/polyfill`



文章开始我们提到，`babel`进行代码转换大致做了两件事，第二件事便和`@babel/polyfill`有直接关系，它的功能就是“缺失特性的修补”。我们放在下面配置文件部分说这个



# 配置文件与`@babel/polyfill`



## 配置文件



除了在命令行中手动通过`--presets`和`--plugins`指定编译过程中使用的`plugin`之外，我们还可以通过配置配置文件的方式来指定插件与预设，比如：

~~~json
// babel.config.json
{
  "plugins": ["@babel/plugin-transform-arrow-functions"]
}
~~~

然后我们命令行通过`babel`程序编译文件时可以省略`--plugins=@babel/plugin-transform-arrow-functions`，通过读取配置文件，效果完全一样。



## `@babel/polyfill`



（polyfill英文释义：填充物）

简单理解就是一旦安装了`@babel/polyfill`，我们就给程序模拟了（提供了）完成的最新版本（ES2015+）的js特性，说白了就是在`babel`处理的文件中添加了新特性的代码实现，同时也提供了按需加载的支持：

我们所使用的 `env` preset 提供了一个 `"useBuiltIns"` 参数，当此参数设置为 `"usage"` 时，`babel`只修补我们所需要的 `polyfill`。使用此新参数后，配置文件如下：

~~~json
// babel.config.json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        },
        "useBuiltIns": "usage"
      }
    ]
  ]
}
~~~

放个官方给的`@babel/polyfill`工作的具体的例子：

Babel 将检查你的所有代码，以便查找目标环境中缺失的功能，然后只把必须的 polyfill 包含进来。示例代码如下：

```js
Promise.resolve().finally();
```

将被转换为（由于 Edge 17 没有 `Promise.prototype.finally`）：

```js
require("core-js/modules/es.promise.finally");

Promise.resolve().finally();
```



# babel与webpack的关系



首先两者是两个完全独立的开源项目，前者由上文我们已经非常明确了，说白了就是一个进行代码转换（降级&特性修补）的编译器；而后者更不用说了，很有名的构建工具。

不过我们不从`webpack`作为一个打包器的角度去看待，而重点在于`webpack`提供了`loader`机制，即在代码打包之前对其进行“预处理”，代码经过`loader`处理的过程就是把代码进行“加工”的过程。而考虑`babel`，作为一个编译器，不就是一个对代码进行加工的工具么！！

所以为了让我们在使用`webpack`进行打包时去调用`babel`进行代码的处理，自然就需要借助一个`loader`，即`babel-loader`，说白了`babel-loader`对代码进行转换的逻辑就是对`@babel/core`中编译方法的包装，`babel-loader`就是一个连接`webpack`与`Babel`的中间桥梁。

2023.4.18，`babel-loader`的源码看了半天，没搞很明白，暂时放弃了，主要原因不是因为源码本身多复杂的逻辑（只不过是一个`babel`与`webpack`链接的中间桥梁罢了），而是在于`webpack`给`loader`提供的各种api没搞懂，一会`this`一会`this.async`，等以后杀回来看吧。

但是我非常清晰的了解了编写一个`loader`该具备的代码结构：

（前置知识：`loader`就是一个函数，接受传来的文件内容，返回处理后的文件内容）

~~~js
let babel;
try {
  // babel即为`@babel/core`所提供的代码转化工具
  babel = require("@babel/core");
} catch (err) {
  if (err.code === "MODULE_NOT_FOUND") {
    err.message +=
      "\n babel-loader@9 requires Babel 7.12+ (the package '@babel/core'). " +
      "If you'd like to use Babel 6.x ('babel-core'), you should install 'babel-loader@7'.";
  }
  throw err;
}

// 首先模块暴露的应该是一个工厂函数的调用，因为我们返回的loader函数是要根据入参进行定制的，所以不可能写死为一个函数
module.exports = makeLoader();

// 然后剩下的逻辑大头就是编写makeLoader了，但是对于babel-loader来说，文件转化的过程只是对`@babel/core`中方法的包装，最难的逻辑应该在于获取配置参数（手动传参、配置文件传参、webpack传参数...我猜的）
function makeLoader(callback) {
  const overrides = callback ? callback(babel) : undefined;

  // makeLoader返回值很明确，就是一个函数，即我们的loader函数
  return function (source, inputSourceMap) {
    const callback = this.async();

    loader.call(this, source, inputSourceMap, overrides).then(
      args => callback(null, ...args),
      err => callback(err),
    );
  };
}
~~~

