# package.json字段



## 包的描述性配置

### name（必须属性）

表示发布后的包名

至于以`@xxx/`开头的`name`字段，表示这是一个作用域包，原因是为了防止包名重复的一种解决方案，类似于创建了一个命名空间，不同的命名空间，可以使用相同的包名，作用域的命名不是随便起的，只有两种可以使用：自己的用户名、自己创建的组织名。举个具体的例子，`@vue/cli`这个包说明使用了`vue`这个`npm`账号或者组织发布了该包。



### version（必须属性）

表示包的版本号。

如果项目是为发布npm包，则必须包含此字段。如果是普通的项目，则此字段是可选的。

每次发布的`version`，必须是唯一的，之前发布的时候没使用过的。



### description

值即对此包的概述。



### keywords

功能类似于`description`，npm平台上展示的关于此包的一些关键词

~~~json
"keywords": [
  "vue",
  "react",
  "next"
]
~~~



### homepage / repository

项目的官网主页地址 / 项目的源码地址。

也算是描述性信息罢了。



### author

作者信息。

~~~json
"author": {
    "name": "leon",
    "email": "582104384@qq.com",
    "url": "https://wangxiaokai.vip"
}
~~~



### contributors

协作者信息。

格式是一个对象数组。对象内容和`author`一致。

```json
"contributors": [{
    "name": "hanmeimei",
    "email": "hanmeimei@qq.com"
},{
    "name": "lihua",
    "email": "lihua@qq.com"
}]
```



## 功能性配置

### files

files是一个文件数组，描述了将软件包作为依赖项安装时要包括的条目。如果在数组里面声明了一个文件夹，那也会包含文件夹中的文件。某些特殊文件和目录也被包括或排除在外，无论它们是否存在于文件数组中。

简而言之，别人下载我们的包所包含的文件。

比如`dance-ui`里只设置了`dist`文件夹

~~~json
"files": [
  "dist"
],
~~~



### main

别人使用npm包时，如果通过`require()`的方式引入包。那么就会查看`main`字段，作为包的主入口文件。



### module

性质等同于`main`字段。`module`用于ES6规范的模块，只要支持ES6，会优先使用`module`入口。换句话说别人通过`import`的方式引入我们的包时，实际上寻找的模块就是`module`字段指定的文件。



### bin

工具性质的npm包，一定有`bin`字段，对外暴露脚本命令。

比如`webpack-cli`为我们在命令行提供了`webpack`命令进行打包，看了下它的`package.json`：

~~~json
"bin": {
	"webpack-cli": "./bin/cli.js"
},
~~~



### scripts

太熟了，就是配置一些项目操作的自动化脚本。



### dependencies、devDependencies、peerDependencies

dependencies非常熟了，就是我们项目安装的直接依赖。

devDependencies与dependencies的区别就是dependencies里的依赖最终是作为项目代码的一部分（有助于功能的实现）；而devDependencies里的依赖是参与项目代码实现的，比如webpack这种构建工具就属于项目的devDependencies，以为它的作用是对开发完成的项目代码进行后期处理。

peerDependencies感觉就是为了服务于发包的，就比如我们的这个库要发包，然后可以通过peerDependencies给安装我们包的用户一些提示，比如我们的包配置如下：

~~~json
"peerDependencies": {
    "react": ">=16.8.0"
}
~~~

那么安装我们包的用户的项目如果react的版本不满足要求，控制台就会给警告。这也就是我们安装一些依赖时控制台经常会报peerDependencies警告的原因，就是因为我们安装的这个包，他本身对一些依赖有版本要求，我们项目里的依赖版本不符合要求。



### types

项目如果是用`TypeScript`写的，则需要`types`字段，对外暴露相关的类型定义。比如`dance-ui(ts编写的react组件库)`项目：

~~~json
"types": "./dist/index.d.ts",
~~~

猜测这就是某些开源库对类型支持友好的原因吧（因为导出了`.d.ts`文件）



### workspaces

对于monorepo架构的项目（现在的工具库主流架构，不知道的朋友建议去补补课，很有意思），有必要配置`workspaces`指明所有子项目的工作目录（子项目根目录），这样应该才支持各个子项目之间作为依赖互相安装使用。这个配置毕竟不是个性化配置，子项目有啥配啥就完事了，反正肯定就能保证项目的正常工作了，比如`packages`文件夹下的所有（直接）子文件夹都是子项目（支持使用通配符，`*`表示直接文件夹，不包含嵌套文件夹）：

~~~json
"workspaces": [
  "packages/*"
],
~~~



## 发布相关的配置

### private

`private`和发布npm包相关。

当`private: true`时，npm会拒绝发布当前项目。这是防止意外发布个人仓库的一种保护方式。

看了下，公司里的业务项目都设置了这个字段为`true`。（发布出去怕是会犯法😂）

我试了一下，`private`设置为`true`之后即使设置了`publishConfig`字段是不生效的，一些文档（官方）与文章中写到`publishConfig`可以搭配`private:true`然后成功发包，我不理解啊！chatgpt很肯定的告诉我：`private`设置为`true`之后是铁定发不出去包的，他没摇摆，我很感动。



### publishConfig

这个配置项是一个对象，里面有`registry`、`access`、`tag`三个比较常用（我感觉够用）的字段

* `registry`：对应我们的包的下载地址（npm源），我们发包的时候就会把包发到这个源里，比如我们可能因为嫌npm慢然后把`registry`设置为国内的一些镜像源，这个字段不设置的话应该默认就是`https://registry.npmjs.org/`（不然怎么会发到npm上呢哈哈）
* `access`：默认值就是`public`，表示我们的包是开放的，大家都可以下载，然后还有个取值`restricted`，我试着发布了一下，失败了，好像要求包必须是一个作用域包（作用域包相关看name字段），感觉这个`access`和`registry`一样，都用默认值就完事了（发包的初衷不就是发布到npm，并且让所有人都能用么😁）
* `tag`：默认值是`lastest`，表示最新，说白了这玩意有点像`git commit`的`-m`，登录到npm查看发包历史，每次发布都能看到有个`tag`字段。



## 跟三方库相关的配置

、

### sideEffects

`sideEffects`格式：`boolean | string[]`。

`sideEffects: false`用于告知打包工具（webpack），当前项目`无副作用`，可以使用`tree shaking`优化。

`sideEffects`的值，也可以是一个文件路径组成的数组。告知哪些文件`有副作用`，不可以使用`tree shaking`优化。

```json
"sideEffects": [
    "a.js",
    "b.js"
]
```

并且，由于`tree shaking`只在`production`模式生效，所以本地开发会一切正常，生产环境很难及时发现这个问题。

当然， 样式文件使用`"import xxx;"`的方式引入，会进行保留。

上面是这篇[文章](https://juejin.cn/post/7108985001529573412)里的描述，但是原文有一定的错误，已经进行了修正，并且我谈下我对`sideEffects`的理解：生产模式下，`webpack`打包时会开启`tree-shaking`，本质上就是`webpack`对程序打包的一种优化手段，但是`tree-shaking`要保证程序的正确运行为前提，所以一段代码该不该被`tree-shaking`掉，本身有一定的判断逻辑，所以`sideEffects`字段就是辅助`webpack`去进行`tree-shaking`的，如果设置为`false`，就表示项目里没有那种一定存在副作用的代码文件，所以可以走`webpack`本身的`tree-shaking`逻辑，但是如果`sideEffects`里面指定了一些存在副作用的代码文件，那么就说明这些文件如果被引用，即使`webpack`本身的逻辑判断他们没有副作用，可以被删除掉，那么也会因为`sideEffects`的配置而被保留，所以算是一个`webpack`进行`tree-shaking`的辅助配置。



### 其它

还有一些如babel、eslint、gitHooks等相关的配置，作者现在还没用到，感兴趣的参考这篇[文章](https://juejin.cn/post/7023539063424548872)



# 发包

这不是我们的重点，但也简单说一下，方便刚入坑的同学自己瞎鼓捣试试😂。

这就比较简单了，对于拥有`package.json`文件的项目（或者不是项目...），`npm login`登陆上我们的`npm`账号后，执行`npm publish`就完事了，完全依据我们配置的`package.json`，就把代码上传到`npm`上了，然后别人就可以安装使用了。



