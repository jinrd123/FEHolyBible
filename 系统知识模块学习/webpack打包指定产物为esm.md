webpack.config.js:

~~~js
const path = require("path");

module.exports = {
  mode: "development",
  entry: "xxx", // 相对此配置文件位置的相对路径，打包入口文件
  output: {
    filename: "xxx.js", // 打包产物名
    path: path.resolve(__dirname, "output"), // 打包产物所属文件夹
    library: {
      type: "module", // 指定打包产物的模块化标准为esm
    },
  },
  experiments: {
    outputModule: true, // webpack对于输出esm产物的支持属于实验阶段，所以这里要配置，表示启用实验特性
  },
  module: {
    rules: [
      // 配置一个rules，用babel简单处理一下js代码
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env"],
          },
        },
      },
    ],
  },
};
~~~



打包入口文件：

~~~js
export { default as Button } from "./button";

export { default as Tab } from "./tab";
~~~



使用打包后的产物：

~~~js
import { Button } from "xxx(打包产物文件)"; // 在打包产物中引入Button
~~~

