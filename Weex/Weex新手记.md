# Weex 新手记
## 前言
上周五 leader 跟我说了一下，想让我转大前端，周一让前端组长跟我聊一下，当时我内心还是比较兴奋的，因为这跟我最开始对自己的定位是完全一致的，但后续做了一些后端的东西后，发现自己对后端也有感觉，不过实话实说都还在比较浅薄的层面上。

上周末我想了两天，发现自己确实得在大前端这块上继续“抛头颅，洒热血”，不单是再精进一些之前已掌握的知识，更是拓展了自己在稍微擅长的一个领域站得更稳，刚好最近负责的产品有往“跨平台”的开发模式上转，需要提前做些调研工作，再跟上最近团队人员的变动，新来的同学有意愿切入 Weex 跨平台框架。

当然，对于我个人而已，我完全不喜欢任何非原生的东西，甚至目前到了用别人的库都觉得“怪异”，今年年初的时候把当时最火的两个跨平台框架“小程序”和 “React-Native” 都摸了一遍，也都把 demo 写了出来，但就仅仅从 demo 这个级别出发，当时就已经非常的厌恶各种跨平台的开发框架了，总有些说不出的“怪异”之处，但也不能说它们毫无是处，在编写 demo 的过程中，很多基础组件上手就用，比 Native 不管是从速度上还是便利上都大大提升了不只一两倍这么简单。

经过一周短暂的、磕磕碰碰的学习，也把今年学习的第三个目前也十分火爆的跨平台框架—— Weex 写出了 demo，其中大部分思路来源于 Weex 的官方文档和教学视频。如果你感兴趣的话，可以扫描下方二维码进行试玩：

<img src="https://i.loli.net/2018/11/24/5bf94c21abf1f.png" alt="1543064551.png" title="1543064551.png" />

## Weex 简单介绍
对于 Weex 的介绍在网上已有大量的讲解，在此用自己的话可以总结如下：
* Weex 写起来很爽，前提是：对动画无要求；对交互无要求；对性能无要求；业务逻辑不复杂。
* 如果你司技术栈 `Javascript` 为主体且依赖 `Vue`，排除第一条后，上 Weex 几乎没有悬念，毕竟到现在 Weex 都被认为是 `Vue-Native`。
* 如果你司已经沉淀出了大量 Native 经验，上 Weex 几乎可以认定公司要倒（只是比喻 😄 ）；反之，如果你司是 web 主导，想切入 Native 端，满足第一条后，再考虑 React-Native 是否符合自身技术栈，其次再来考虑 Weex。验证我这番话，可以对比 RN 和 Weex 的官方文档和社区，当然 Weex 还是十分的年轻，但和 React-Native 二者正式开源也就前后相隔一年左右。
* 据我所知，目前“极客时间“、”企鹅电竞“等 App 已经是纯 Weex 开发，甚至桔厂”顺风车“也全面拥抱，可见其威力有多大！

## 知识点
该 demo 中运用到的主要相关知识点如下：
* Weex 内置组件：`div`，`text`
* Weex 内置模块：`navigator`，`storage`，`dom`
* Weex custom Component
* `CSS` 基本内容
* `Vue.js` 基本内容
* `Javascript` 基本内容

## 页面展示
### index.html
<img src="https://i.loli.net/2018/11/24/5bf9564943b55.png" height=70% />

### add.html
<img src="https://i.loli.net/2018/11/24/5bf95686d9727.png" height=70% />

### detail.html
<img src="https://i.loli.net/2018/11/24/5bf956a8ae508.png" height=70% />

## 开发过程
Weex 吸收了目前最流行的 MVVM 和面向组件开发的思想，上文中我所说的“爽”就来自于此！举一个例子，`navbar` 组件，编写一个组件相对 Native 来说，真的是又快又爽！在 `<tempalte>` 中写好组件模版代码，在 `<script>` 中写好事件处理、属性定义、生命周期管理，在 `<style>` 中写好 `CSS` 样式布局，想要给别人用，直接拖走这个 `.vue` 文件即可完事（当然 native 也是一样）。我觉得能有如此便利，多亏 CSS，在 native 实现一个功能可能使用 CSS 只需要两三行即可完事！

### `navbar` 组件
```vue
<template>
  <div class="navbar">
      <!-- v-if 判断是否需要展示返回 icon -->
      <text v-if="showBack" class="iconfont navbar-icon" @click="onBack">&#xe779;</text>
      <text v-else class="navbar-title"></text>
      <text class="navbar-title">{{ title }}</text>
      <text class="navbar-title"></text>
  </div>
</template>

<script>
const navigator = weex.requireModule('navigator')
export default {
  name: 'navbar',
  props: {
    showBack: {
      type: Boolean,
      default: true
    },
    title: {
      type: String,
      required: true
    }
  },
  beforeCreate () {
    const domModule = weex.requireModule('dom')
    domModule.addRule('fontFace', {
      'fontFamily': 'iconfont',
      'src': "url('http://at.alicdn.com/t/font_933576_ji32n9fdyki.ttf')"
    })
  },
  methods: {
    onBack () {
      navigator.pop({
        animated: 'true'
      })
    }
  }
}
</script>

<style scoped>
.iconfont {
    font-family: iconfont;
  }
.navbar {
  height: 88px;
  background-color: #50e3a4;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding-left: 20px;
  padding-right: 20px;
}
.navbar-title {
  font-size: 32px;
  color: #fff;
}
.navbar-icon {
  color: #fff;
  font-size: 36px;
}
</style>
```

## 相关注意点
### 安装 Weex 工具包
`npm install weex-toolkit -g`

### 从零开始创建 Weex 工程
`weex create awesome-app`

在创建工程的过程中，会提示一些关键信息，比如作者、是否使用 `vue-router`，`ESLint` 等等，根据提示等待即可。

### 添加 iOS 工程
`weex platform add ios`

### 构建 js bundle
`weex run build` 

在 dist 文件夹下拿到对应的 js bundle 文件。

### 切换显示
在工厂目录下执行 `npm start` 后，会在浏览器打开一个“套壳”的页面，有很多不需要的元素，如果不需要的话，可以这么做：

* 假设执行 `nmp start` 后，打开的地址为：`http://172.20.10.4:8081/web/preview.html?page=index.js`
* 把地址改为：`http://172.20.10.4:8081/index.html`，这样就去除掉了多余不需要的元素了，页面变得十分干净

### 新增页面
新增页面后，此时如果通过浏览器直接输入地址访问会 404，因为此次 build 出来的资源文件中并未包含我们新增的页面，需要重新执行 `npm start` 进行重新构建。

### flex-direction
决定你的页面布局主要方向，是**row**（水平）还是**column**（垂直布局）。

### align-items
决定父容器中的元素在水平方向上的布局，想要居中则设置为 `center`。

### align-content
决定父容器中的元素在垂直方向上的布局，想要居中则设置为 `center`。

### justify-content
决定父容器中的元素在主轴上如何排列，如果想要等分布局，则设置为 `space-around`，左右边距将为中间间隔的一半。

## align-items
决定元素在交叉轴上如何排列

### dist
通过 webpack 打包后生成的 `JS Bundle` 文件都在 `dist` 文件夹下。

### 在模版中，Vue 会把驼峰命名的组件自动转换成短横线连接

### Boolean
在 Weex 中关于 bool 值，本质上为字符串，比如`"true"` 这样才是“真”，`true`这样什么也不是，官方说会在未来版本中进行修复，还有很多类似这种容易引起“差评”的地方。

### Weex SDK
构建出来的 js bundle 直接直接可以拖入工程使用，在 iOS 下，看到的渲染后的页面层级如下：

<img src="https://i.loli.net/2018/11/24/5bf95a719ad28.png" width=70% />

查看 WeexSDK，可以看到基本上把原生组件都按照 Weex 支持的格式封装了一遍，所以加入跨平台框架后，app 体积不上升是不可能的，只不过得看用什么个优化方法了（删删删哈哈哈～）

## 总结
本次 Weex demo 的练习，让自己对 Weex 和 Vue 都有了一个直观的感受，到现在给我印象最深刻的不是 Weex 而是 Vue，感触良多。对原生开发的喜爱又多了不少，在今天这个时代背景下，求快不求稳，不管怎么说，我还是一个鉴定的原生开发者～