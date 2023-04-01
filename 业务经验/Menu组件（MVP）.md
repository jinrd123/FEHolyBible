# 需求分析

Vue3简单实现一个MVP版本的Menu，需求分析：

* `props`接收一个`menuList`，作为菜单选项列表进行渲染
* 消费组件（父组件）可以通过`v-model`获取到当前被选中的菜单选项的值，即`menuList[index]`
* 消费组件可以通过监听`Menu`组件的`changeMenu`事件来指定在菜单切换时的回调



# 结构搭建

MVP布局的话，单纯使用一个`ul`作为容器，里面每一项用`li`渲染即可，但这里出于多种考虑，外面再包一层`div.menu-container`，这样可以：

* 拓展导航栏的布局灵活性，易于拓展。例如我们一个横向的菜单导航，左边可能几个选项，右边也有几个选项，这时候可以在`div.container`里面再添加一个`ul`，实现选项集合`ul`的左右布局
* 作为一个vue组件，包裹一个`div`在开发者工具中看dom结构也更加清晰
* ...

并简单添加样式，让`li`标签一行显示，并增加一定的间距

~~~vue
<template>
  <div class="menu-container">
    <ul class="menu-ul">
      <li>导航选项1</li>
      <li>导航选项2</li>
      <li>导航选项3</li>
      <li>导航选项4</li>
    </ul>
  </div>
</template>

<style scoped lang="less">
.menu-container {
  .menu-ul {
    display: flex; // 布局属性: li标签一行排列
    column-gap: 50px; // 样式属性: 控制选项间隙，在flex项目与项目之间添加间隙
    li {
      list-style-type: none; // 样式属性: 去除li标签的默认小圆点样式
      background-image: linear-gradient(90deg, red, green); // 样式属性: 方便观察
    }
  }
}
</style>
~~~

效果：

![image-20230401174348409](./记录img/Menu静态布局.png)



# 实现`props`接收渲染列表



`<script>`中添加一个`props`配置项，我们图方便直接用`default`给一个默认值，然后`<template>`中也`v-for`替换渲染数据

~~~vue
<template>
  <div class="menu-container">
    <ul class="menu-ul">
      <li v-for="(menuItem, index) in menuList" :key="index">{{ menuItem }}</li>
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";

export default defineComponent({
  name: "Menu",
  props: {
    menuList: {
      type: Array as PropType<string[]>,
      default() {
        return [
          "产品&市场",
          "技术&研发",
          "管理&职场",
          "活动&生活",
          "其他",
        ];
      },
    }
  },
  setup(props) {
    return {};
  },
});
</script>
~~~

效果：

![image-20230401175848060](./记录img/Menu数据动态渲染.png)

但这里有个坑，如果在`setup`中通过`props.menuList`的形式访问`props`属性就会报错，我不知道具体是不是因为打包工具的原因，用`vue-cli`创建的基于`webpack`的项目上面的代码会TS报错：`Property 'menuList' does not exist on type 'Readonly<LooseRequired<Readonly<ExtractPropTypes<readonly string[] | Readonly<ComponentObjectPropsOptions<Record<string, unknown>>>>> & { ...; }>>'`，但是基于`vite`就没事。报错时如果想要在`js`代码中访问`props`，暂时我只想到了`(props as any).menuList`。针对这个错误真的很无奈...





# 添加`v-model`支持    



首先`Menu`组件中需要添加一个`props`属性，这个属性即为消费组件（父组件）中`v-model`绑定的值，即父组件中

`App.vue`：

~~~vue
<template>
	// 即我们希望通过activeMenuInfo变量来获取Menu组件中选中的标签值，v-model:activeMenuItem，这里的activeMenuItem对应Menu组件中的props.activeMenuItem
  <Menu v-model:activeMenuItem="activeMenuInfo" :menuList="menuList"/>
</template>

<script setup lang="ts">
import Menu from "@/components/Menu.vue";
import {ref} from "vue";
  
const menuList = ["产品&市场", "技术&研发", "管理&职场", "活动&生活", "其他"];
const activeMenuInfo = ref(menuList[0]); // activeMenuInfo初始化值理论上应为传给Menu组件的渲染列表的第一项
</script>
~~~



`Menu.vue`:

~~~js
...
props: {
		...
    activeMenuItem: { // 添加activeMenuItem
      type: String
    }
  },
},
~~~

同时，给`li`标签绑定`Menu`导航切换时的回调函数`changeMenu`，并传入`index`，这样可以在`changeMenu`函数体中通过`props.menuList[index]`访问选中的新导航的值。

`<Menu />`组件中`emit`数组中添加一个`"update:activeMenuItem"`事件，只需要调用`emit("update:activeMenuItem", value)`即相当于把父组件中通过`v-model:activeMenuItem="xxx"`绑定的`xxx`变量修改为`value`，这里`emit`是从`setup`函数第二个对象参数中解构出来的：

~~~vue
<template>
  <div class="menu-container">
    <ul class="menu-ul">
      <li 
        v-for="(menuItem, index) in menuList" 
        :key="index"
        @click="changeMenu(index)"
      >
        {{ menuItem }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";
export default defineComponent({
  name: "Menu",
  props: {
    menuList: {
      type: Array as PropType<string[]>,
      default() {
        return [
          "产品&市场",
          "技术&研发",
          "管理&职场",
          "活动&生活",
          "其他",
        ];
      },
    },
    activeMenuItem: { // 这个props属性即为父组件中 v-model: 冒号后面的字段，同时也是emit事件 update: 冒号后面的字段
      type: String
    }
  },
  emits: ["update:activeMenuItem"],
  setup(props, {emit}) {
    const changeMenu = (index: number) => {
      emit("update:activeMenuItem", props.menuList[index]);  // 切换导航时修改父组件中v-model绑定的变量为props.menuList[index]
    }
    return {
      changeMenu
    };
  },
});
</script>
~~~



这样已经完成了`Menu`组件对`v-model`的支持，可以在父组件中通过`watch`函数监听来测试`v-model`绑定的变量的变化，点击导航后`newValue`和`oldValue`都正常访问，功能实现是没有问题的。

当然一般都会给选中的导航添加一些样式，所以我们在`Menu`组件中维护一个被激活导航的标识变量`activeIndex`，`v-for`遍历生成`li`时给`index === activeIndex`的`li`添加样式。



# 实现父组件对导航切换事件的监听

很简单，只需要在`Menu`组件中`emits`数组中添加一个`changeMenu`事件，然后在`Menu`组件内部的导航切换回调中执行`emit("changeMenu")`，即可触发父组件中的回调函数。



# `<Menu />`最终代码

[gitHub](https://github.com/jinrd123/MenuDemo)

`Menu.vue`:

~~~html
<template>
  <div class="menu-container">
    <ul class="menu-ul">
      <li 
        v-for="(menuItem, index) in menuList" 
        :key="index"
        @click="changeMenu(index)"
        :class="{
          'active-li': index===activeIndex
        }"
      >
        {{ menuItem }}
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, ref} from "vue";
export default defineComponent({
  name: "Menu",
  props: {
    menuList: {
      type: Array as PropType<string[]>,
      default() {
        return [
          "产品&市场",
          "技术&研发",
          "管理&职场",
          "活动&生活",
          "其他",
        ];
      },
    },
    activeMenuItem: {
      type: String
    }
  },
  emits: ["update:activeMenuItem", "changeMenu"],
  setup(props, {emit}) {
    // 选中导航的标识变量
    const activeIndex = ref(0);
    // 切换导航的回调
    const changeMenu = (index: number) => {
      activeIndex.value = index;
      emit("update:activeMenuItem", props.menuList[index]);
      emit("changeMenu");
    }
    return {
      changeMenu,
      activeIndex
    };
  },
});
</script>

<style scoped lang="less">
.menu-container {
  .menu-ul {
    display: flex; // 布局属性: li标签一行排列
    column-gap: 50px; // 样式属性: 控制选项间隙，在flex项目与项目之间添加间隙
    li {
      list-style-type: none; // 样式属性: 去除li标签的默认小圆点样式
      background-image: linear-gradient(
        90deg,
        red,
        green
      ); // 样式属性: 方便观察
      &.active-li {
        background-image: linear-gradient(
          270deg,
          red,
          green
        );
      }
    }
  }
}
</style>
~~~

`App.vue`：

~~~html
<template>
  <Menu 
    v-model:activeMenuItem="activeMenuInfo" 
    :menuList="menuList"
    @changeMenu="test"
  />
</template>

<script setup lang="ts">
import Menu from "@/components/Menu.vue";
import {ref, watch} from "vue";
const menuList = ["产品&市场", "技术&研发", "管理&职场", "活动&生活", "其他"];
const activeMenuInfo = ref(menuList[0]); // activeMenuInfo初始化值理论上应为传给Menu组件的渲染列表的第一项
watch(activeMenuInfo, (newValue, oldValue) => {
  console.log(newValue, oldValue);
})
const test = () => {
  console.log("父组件中切换导航的回调函数");
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
</style>
~~~

