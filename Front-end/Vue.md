# Vue
## 基础知识
### MVVM 数据绑定
在学习 `Vue` 的过程中，非常好奇 `Vue` 自身的数据绑定模式是否与我之前在 iOS 上接触到的数据绑定认识一致，然后又在资料中发现了 `Object.defineProperty()` 和 `Object.defineProperties` 方法，相当于对一个属性进行了 `setter` 和 `getter` 的监听，然后根据监听结果重新更新元素，大致如下所示：

```html
<html>
  <head>
  </head>
  <body>
    <span id="span"></span>
    <script>
      var obj = {
        pwd: "123123"
      };
      Object.defineProperty(obj, "userName", {
        get: function () {
          console.log("get init")
        },
        set: function (newValue) {
          console.log("set init")
          document.getElementById("span").innerText = newValue
        }
      });
    </script>
  </body>
</html>
```

看到这段代码后，瞬间就大彻大悟！其实也不是什么特别高深的内容（当然还是有其它值得学习的地方～），而且这么做耦合度十分的高，就目前的学习内容来看，`Vue` 中对 **组件化** 的核心思路跟之前在 iOS 中流程还是一致的～

## 构建
### 生成 package.json 文件（需要手动选择配置）
`npm init`

### 生成 package.json 文件（使用默认配置）
`npm init -y`

### 一键安装 package.json 下的依赖包
`npm i`

### 在项目中安装包名为 xxx 的依赖包（配置在 dependencies 下）
`npm i xxx`

### 在项目中安装包名为 xxx 的依赖包（配置在 dependencies 下）
`npm i xxx --save`

### 在项目中安装包名为 xxx 的依赖包（配置在 devDependencies 下）
`npm i xxx --save-dev`

### 全局安装包名为 xxx 的依赖包
`npm i -g xxx...`

### 自定义执行命令
也就是会运行 `package.json` 中 `scripts` 下的命令：
```shell
npm run xxx
```

## 懒加载
H5 页面也可以像之前原生中的“懒加载”思想一致，等到需要真的需要使用这个组件时，再对其进行渲染，如果你跟我一样使用 `vue-router`，则可以这么写： 

```js
import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'

Vue.use(Router)

export default new Router({
  routes: [
    {
      // 该页面为直接加载
      path: '/',
      name: 'home',
      component: Home
    },
    {
      // 该页面为懒加载
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (about.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: function () {
        return import(/* webpackChunkName: "about" */ './views/About.vue')
      }
    }
  ]
})
```

## Vue 的一些属性描述
### `name`
只有作为**组件选项**时起作用。允许组件模版递归地调用自身，注意，组件在全局用 `Vue.component()` 注册时，全局 ID 自动作为组件的 name；指定 `name` 选项的另一个好处是便于调试。有名字的组件有更友好的警告信息。另外，当在有 `vue-devtools`，未命名组件将显示成 `<AnonymousComponent>`，这很没有语义。通过提供 `name` 选项，可以获得更有语义信息的组件树。

## Props
### `Array`
如果我们要传递一个字符串数组给子组件，错误的传递方式：

```html
<nav-header item-names="['笔记', '海洋', '书写', '通知', '我']"></nav-header>
```

正确的传递方式：

```html
<nav-header v-bind:item-names="['笔记', '海洋', '书写', '通知', '我']"></nav-header>
```

vue 官网上是这么说的，
> 即便数组是静态的，我们仍然需要 `v-bind` 来告诉 Vue，这是一个 `JavaScript` 表达式而不是一个字符串。

###m mint-UI 是怎么实现下拉刷新的呢？
简单来说获取下拉手势后通过 `transform` 来做动画偏移。