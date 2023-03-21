# 结论

场景：`node`环境下运行一个`js`脚本：`node xxx/xxx/xxx/脚本名.js`

* `__dirname`：`node`命令要执行的脚本在计算机中的绝对位置，即`脚本名.js`脚本文件所在的位置

* `process.cwd()`：执行`node`命令所在计算机中的绝对位置，即`node xxx/脚本名.js`这个命令是在哪执行的



# 举例

随便建个`test`文件夹，内部结构如下：

~~~
test/
└── dir_top/
    └── dir_mid/
    		└──test.js
~~~

`test.js`：

~~~js
console.log(__dirname);

console.log(process.cwd());
~~~

在`test`文件夹内打开终端，即当前位置`/xxx/xxx/test`

执行命令：

~~~bash
node ./dir_top/dir_mid/test.js
~~~

结果：

~~~js
console.log(__dirname); // 输出：/xxx/.../test/dir_top/dir_mid，即脚本文件test.js所在的位置

console.log(process.cwd()); // 输出：/xxx/.../test，即node命令执行的位置
~~~

