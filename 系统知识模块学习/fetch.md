# 背景

`axios`是基于`XMLHttpRequest`对象进行了`promise`封装，通过传统`Ajax`实现数据请求：

~~~js
// 1. 创建一个xhr对象
let xhr = new XMLHttpRequest();
// 2. 设置请求方式和请求地址
xhr.open('get', 'http://abc');
// 3. 发送请求
xhr.send();
// 4. 监听load事件获取响应结果
xhr.addEventListener('load', function() {
  console.log(JSON.parse(xhr.response));
})
~~~



**`fetch`（`window.fetch`）是一种HTTP数据请求的方式，是`XMLHttpRequest`的一种替代方案，是原生js提供的。**



# 原生fecth的基本使用



* 如果`fetch()`只接收一个url字符串参数，默认为`get`请求，返回一个Promise对象
* 如果需要设置get请求的参数，直接拼接到url字符串上



~~~js
fetch('http://abc').then(res => {
  // 直接得到的是一个Response对象，需要调用json方法之后拿到其中的内容
  console.log(res);
  return res.json();
}).then((jsonContent) => {
  // 获取经过res.json()处理之后的数据
  console.log(jsonContent);
}).catch(err => {
  console.log(err);
})
~~~

修改为`async/await`的形式：

~~~js
async function getData() {
  try {
    let res = await fetch("http://abc");
    let jsonContent = await res.json();
    console.log(json);
  } catch(err) {
    console.log(err);
  }
}
getData();
~~~



# Response对象常用属性、方法



~~~js
async function getData() {
	let res = await fetch('http://abc');
  console.log(res);
  console.log(res.ok); // true or false，表示请求是否成功
  console.log(res.status); // http请求状态码
  console.log(res.statusText); // http请求描述信息
  console.log(res.url); // 请求url
  
  let jsonContent = await res.json();
  console.log(jsonContent); // json形式的响应对象
}
getData();
~~~



# fetch使用



`fetch`的第一个参数是`url`，此外还可以接受第二个参数：配置对象，即`fetch(url, options)`用来自定义发出HTTP请求

其中`post`,`put`,`patch`用法类似，如下为`post`



## 配置参数基本格式



~~~js
fetch(url, {
  method: '请求方式，比如：post、delete、put',
  headers: {
    'Content-Type': '请求体的数据格式'
  },
  body: '请求体的数据'
})
~~~



## 发送一个`post`（添加）请求：



~~~js
async function add() {
  let obj = {
    bookname: 'bookjrd',
    author: 'jrd',
    publisher: 'gangangan'
  };
  
  let res = await fetch("http://abc", {
    method: 'post',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(obj);
  });
  
  let jsonContent = await res.json();
  console.log(json);
}

add();
~~~





## fetch封装



~~~js
// 封装http函数目标：
/*
	1. 发送get请求、delete请求
	http({
		method: 'xxx',
		url: 'xxx',
		params: { ... }
	})
	
	2. 发送post请求、put请求、patch请求
	http({
		method: 'xxx',
		url: 'xxx',
		data: { ... }
	})
*/

async function http(options) {
  let { method, url, params, data } = options;
  
  // params处理成key=value&key1=value1的形式
  if(params) {
    // 固定写法：
    let str = new URLSearchParams(params).toString();
    
    url += '?' + str;
  }
  
  let res;
  
  // 对于有data（请求体）的请求，需要写完整的headers...；如果没有data，不需要设置headers，直接发送
  if(data) {
    res = await fetch(url, {
      method,
      headers: {
        'Content-Type': 'application/json' // 针对请求体数据格式为json的情况
      },
      body: JSON.stringify(data);
    })
  } else {
    res = await fetch(url);
  }
  
  return res.json();
}
~~~



Plus：以前用axios发请求时使用`axios()`方法，它的配置对象参数里有`params`和`data`两个子对象。这俩概念一直迷迷糊糊的，不知道到底作为http请求的哪一部分进行传递，前两天学习了一些http相关的知识，感觉现在可以理解了：

* `data`就是http报文的请求体部分（请求行+请求头+请求体）
* `params`就是http请求的一些参数，它是属于`url`的。

**所以上面封装时针对用户传来的参数，`params`参数是需要稳定拼接在`url`中的，然后`data`数据作为`http`报文的请求体部分，本来就是可有可无，没有的时候比较多，所以根据有无进行逻辑处理即可。**