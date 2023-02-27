# 单个vue项目支持移动端&pc端的方案

接手了部门的一个新项目，这个项目支持pc端或移动端打开。

它的一个适配逻辑是根据`window.navigator.userAgent`属性值对客户端为移动端或者pc端进行判断，然后走两套不同的路由。说白了通过路由把两个独立（mobile or pc）的vue项目“杂糅”在这同一个项目中了。

`userAgent`：用户代理，简称UA，包括了用户设备相关的信息，比如操作系统信息、浏览器信息（版本、引擎...）、电子设备信息...，我们前端通过`window.navigator`对象可以访问`userAgent`，对应服务端可以通过http请求头的`userAgent`字段访问。

项目里的`@/router/index.ts`：

~~~typescript
import {createRouter, createWebHistory, createWebHashHistory} from "vue-router";
import {routesMobile} from './mobile';
import {routesPc} from './pc';
const Adaptive = /(phone|pad|pod|iPhone|iPod|ios|iPad|Android|Mobile|WebOS|Symbian|Windows Phone)/i; // 由此推断应该是：用户使用移动设备访问的话navigator.userAgent中会包含这些字段
const routes = (Adaptive.exec(navigator.userAgent)) ? routesMobile : routesPc; // 动态选择routes对象
const router = createRouter({
    history: createWebHistory(),
    routes,
});

export default router;
~~~



项目结构示意：

~~~
/src:
		/pc
				/views
				Home.vue
				...
		/mobile
				/views
				Home.vue
				...
		/router
				index.ts
				mobile.ts
				pc.ts
~~~

