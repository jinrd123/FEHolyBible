# 应用场景

通过vue-cli创建的基于webpack构建工具的vue项目要想自定义的进行webpack配置，需要修改`vue.config.js`中导出的配置对象的`chainWebpack`属性：

~~~js
const {defineConfig} = require('@vue/cli-service')

module.exports = defineConfig({
  ...
  chainWebpack: config => {
    ... // 基于config对象，进行各种api的链式调用 ——> 配置好了webpack配置对象
  }
  ...
})
~~~

## 痛点分析

webpack-chain（[github](https://github.com/Yatoo2018/webpack-chain/tree/zh-cmn-Hans)）这个开源库的目标就是提供一种新的方式创建和修改webpack配置，网上有不少地方吹这种链式写法本身带来的方便性，但是我怎么想都不是很接受，我感觉甚至为了写一个js对象还要承受更多的心智负担去学习这种写法的各种知识得不偿失。存在即合理，而且还被vue-cli等这么多项目所采纳，仔细思考了一下，我认为它的优势（所解决的webpack传统配置对象的痛点）：

* 相当于在输出`webpack`配置对象之前增加了一个编程式（写js逻辑）的预处理过程，即给`webpack`提供配置对象这个事情从“纯运行时”变成了“编译时+运行时”，这样做的好处就不言而喻了，就是编程本身带来的灵活性。
* 对于vue-cli来说，本身已经内置了一个webpack的配置，如果接收用户输入再进行输入对象与原有对象的基于对象的合并操作的话，想来也是比较费劲的。所以webpack-chain可以被vue-cli所采纳，即不管是vue-cli内部的默认webpack配置还是用户输入，一直处理webpack-chain提供的config对象，最终转换config对象输出的配置对象（我没看vue-cli的源码，但是我猜是这样的...）



# webpack-chain链式语法



## 常用配置链式语法实例

参考文章：[Webpack-chain 从入门到深入](https://juejin.cn/post/6947851867422621733)

### 1、entry 入口配置

```javascript
// 配置编译入口文件
config.entry('main').add('./src/main.js') 

// 等同于以下 webpack 配置
entry: {
  main: [
    './src/main.js'
  ]
}
```

### 2、output 出口配置

```javascript
// 配置出口文件
config.output
  .path(path.resolve(__dirname, './dist'))
  .filename('[name].[chunkhash].js')
  .chunkFilename('chunks/[name].[chunkhash].js')
  .libraryTarget('umd');

// 等同于以下 webpack 配置
output: {
  path: path.resolve(__dirname, './dist'),
  filename: '[name].[chunkhash].js',
  chunkFilename: 'chunks/[name].[chunkhash].js',
  libraryTarget: 'umd'
},
```

### 3、alias 别名配置

```javascript
// 配置目录别名
config.resolve.alias
  .set('@', path.resolve(__dirname, 'src'))
  .set('assets', path.resolve(__dirname, 'src/assets'))

// 等同于以下 webpack 配置
resolve: {
  alias: {
    '@': path.resolve(__dirname, 'src'),
     assets: path.resolve(__dirname, 'src/assets')）
  }
},
```

### 4、loader 配置新增

```javascript
// 配置一个新的 loader
config.module
.rule('babel')
.test(/\.(js|jsx|mjs|ts|tsx)$/)
.include
  .add(path.resolve(__dirname,  'src'))
  .end()
.use('babel-loader')
  .loader('babel-loader')
  .options({
    'presets':['@babel/preset-env']
  })

// 等同于以下 webpack 配置
module: {
  rules: [
    {
      test: /\.(js|jsx|mjs|ts|tsx)$/,
      include: [
        path.resolve(__dirname,  'src')
      ],
      use: [
        {
          loader: 'babel-loader',
          options: {
              presets: [
                '@babel/preset-env'
              ]
            }
        }
      ]
    }
  ]
}
```

### 5、loader 配置修改

跟新增 loader 不同的是，使用了 tap 方法，该方法的回调参数为 options 即该 loader 的配置选项对象，从而我们可以通过更改 options 对象，从而去更改 loader 配置。

```javascript
config.module
.rule('babel')
.use('babel-loader')
  .tap(options => {
    // 修改它的选项...
    options.include = path.resolve(__dirname,  'test')
    return options
  })
```

### 6、loader 配置移除

```javascript
config.module.rules.clear(); // 添加的 loader 都删掉.

config.module.rule('babel').uses.clear();  删除指定 rule 用 use 添加的
```

### 7、plugin 配置新增

```javascript
// 配置一个新的 plugin
config.plugin('HtmlWebpackPlugin').use(HtmlWebpackPlugin, [
  {
    template: path.resolve(__dirname, './src/index.html'),
    minify: {
      collapseWhitespace: true,
      minifyJS: true,
      minifyCSS: true,
      removeComments: true,
      removeEmptyAttributes: true,
      removeRedundantAttributes: true,
      useShortDoctype: true
    } 
  }
]);

// 等同于以下 webpack 配置
  plugins: [
    new HtmlWebpackPlugin(
      {
        template: path.resolve(__dirname, './src/index.html'),
        minify: {
          collapseWhitespace: true,
          minifyJS: true,
          minifyCSS: true,
          removeComments: true,
          removeEmptyAttributes: true,
          removeRedundantAttributes: true,
          useShortDoctype: true
        }
      }
    )
  ],
```

### 8、plugin 配置修改

跟新增 loader/plugin 不同的是，使用了 tap 方法，且保留了之前配置的选项，更改的选项被覆盖。

```javascript
// 修改插件 HtmlWebpackPlugin
config.plugin('HtmlWebpackPlugin').tap((args) => [
  {
    ...(args[0] || {}),
    template: path.resolve(__dirname, './main.html'),
  }
]);
```

### 9、使用 when 条件进行配置

```javascript
// 1、示例：仅在生产期间添加minify插件
config
  .when(process.env.NODE_ENV === 'production', config => {
    config
      .plugin('minify')
      .use(BabiliWebpackPlugin);
  });

// 2、示例：只有在生产过程中添加缩小插件，否则设置 devtool 到源映射
config
  .when(process.env.NODE_ENV === 'production',
    config => config.plugin('minify').use(BabiliWebpackPlugin),
    config => config.devtool('source-map')
  );
```

### 10、插件移除配置

```javascript
config.plugins.delete('HtmlWebpackPlugin');
```



我感觉理解webpack-chain的大概写法与规则，然后用到的时候去查阅就可以了，而不是死记硬背去熟练掌握webpack-chain的api。

为了能做到“知其然知其所以然”，我翻看了一下webpack-chain的源码，大致了解了一下这种链式调用的底层实现



## 链式语法底层实现原理



### 部分源码

webpack-chain提供的config对象以及他的大部分链式api，其实是对`Map`和`Set`数据结构的封装，大致浏览一下下面的`ChainMap.js`即可，后面咱再细说：

`Chainable.js`：

~~~js
module.exports = class {
  constructor(parent) {
    this.parent = parent;
  }

  batch(handler) {
    handler(this);
    return this;
  }

  end() {
    return this.parent;
  }
};
~~~

`ChainMap.js`：

~~~js
module.exports = class extends Chainable {
  constructor(parent) {
    super(parent);
    this.store = new Map();
  }

  extend(methods) {
    this.shorthands = methods;
    methods.forEach(method => {
      this[method] = value => this.set(method, value);
    });
    return this;
  }

  getOrCompute(key, fn) {
    if (!this.has(key)) {
      this.set(key, fn());
    }
    return this.get(key);
  }

  set(key, value) {
    this.store.set(key, value);
    return this; // set方法返回this，实现支持链式调用
  }
	...
};
~~~

还有`ChainSet.js`，和`ChainMap.js`做的事情类似，就是对`Set`数据结构的封装：

~~~js
module.exports = class extends Chainable {
  constructor(parent) {
    super(parent);
    this.store = new Set();
  }

  add(value) {
    this.store.add(value);
    return this; // set方法返回this，实现支持链式调用
  }
  
  ...
}
~~~



webpack-chain暴露给用户的config对象是怎么利用上面的`ChainMap`的呢：

`Config.js`：

~~~js
module.exports = class extends ChainedMap {
  constructor() {
    super(); // config为顶层对象，他没有parent，这里不用传参
    // 总之吧，config对象的所有属性，都被初始化为ChainMap（或者ChainSet）
    this.entryPoints = new ChainedMap(this);
    this.node = new ChainedMap(this);
    this.plugins = new ChainedMap(this);
    ...
    this.extend([
      'amd',
      'bail',
      'cache',
      'context',
      'devtool',
      'externals',
      'loader',
      'mode',
      'name',
      'parallelism',
      'profile',
      'recordsInputPath',
      'recordsPath',
      'recordsOutputPath',
      'stats',
      'target',
      'watch',
      'watchOptions',
    ]);
  }
	
  ...
  
  entry(name) {
    return this.entryPoints.getOrCompute(name, () => new ChainedSet(this));
  }

  plugin(name) {
    return this.plugins.getOrCompute(name, () => new Plugin(this, name));
  }
	
	...

};
~~~

### `add`方法

以`entry`配置为例：

~~~js
// 配置编译入口文件
config.entry('main').add('./src/main.js') 

// 等同于以下 webpack 配置
entry: {
  main: [
    './src/main.js'
  ]
}
~~~

`config.entry('main')`相当于`config.entryPoints`这个`ChainMap`对象调用了`getOrCompute`方法，即给`config.entryPoints`增加一个`key`为`'main'`，`value`为`ChainedSet`对象的键值对，并且返回这个`ChainSet`对象，`ChainSet`对象调用`add`方法（`add`方法就是对`Set`数据结构`add`方法的封装，最后会返回`ChainSet`对象本身，所以可以继续调用`add`等方法）



### `end`方法

以新增一个`loader`规则配置为例

~~~js
// 配置一个新的 loader
config.module
.rule('babel')
.test(/\.(js|jsx|mjs|ts|tsx)$/)
.include // ---------- 注释如下 ----------
  .add(path.resolve(__dirname,  'src'))
  .end()
.use('babel-loader')
  .loader('babel-loader')
  .options({
    'presets':['@babel/preset-env']
  })
~~~

`include`肯定返回了一个`ChainSet`，然后调用`add`方法后“调用链终端”还是这个`ChainSet`，因为这个`rule`规则中后面还有一个`loader`，所以调用链上调用`end`方法，即返回上一级的`ChainMap`或者`ChainSet`，详见上面`Chainable.js`



### `.属性(值)`

对于 `ChainMap`，有这样一种简化的写法，官网称之为**速记写法**:

```js
devServer.hot(true);

// 上述方法等效于:
devServer.set('hot', true);
```

原理就是`devServer`是继承了`ChainMap`类的对象，创建`devServer`时调用了`ChainMap`的`extend`方法：

~~~js
extend(methods) {
  this.shorthands = methods;
  methods.forEach(method => {
    this[method] = value => this.set(method, value); // 相当于给ChainMap对象挂载了一堆属性方法
  });
  return this;
}
~~~