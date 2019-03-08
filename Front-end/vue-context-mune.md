# 使用 Vue 实现 Context-Menu 的思考与总结
## 简介
先来看最终成果：

![context-menu.jpg](https://i.loli.net/2019/03/07/5c80d8bf75fd2.jpg)

操作逻辑为：
* 点击 `...` 弹出 `context-menu`;
* 点击非 `context-menu` 区域，隐藏 `context-menu`；
* 点击 `context-menu` 中的任何一个选项，隐藏 `context-menu`；

## 思考
项目是基于 `vux` 做的，本想着偷懒直接在 `vux` 库翻组件用，但看了一圈下来，居然这么通用的组件在 `vuex` 中没有！接着又去翻开源的解决方案，看了几个库还算 ok，但此时前端小哥来了，说实现这个菜单不需要用到这么重的东西，直接写就行了。

当时我的脑海中在思考了把 `context-menu` 封装成一个 `component`，通过数据配置的方式动态拓展菜单选项。但没想到前端小哥直接给我干了回来，没必要进行封装，这个组件对页面依赖性太强，就算封装完了下次也不一定能直接用，PM 的思路十分清奇。

所以，最后的做法就直接硬上了。

## 实现
### 调整操作逻辑
该页面是一个通俗意义上的列表展示页，使用了 `vux` 的 `swipeout` 表单组件，给用户提供了侧滑操作，需要把原先写好的侧滑功能删除。

### 调整 UI
在调整 UI 的过程中我感到了 CSS 满满的恶意，当然说是这么说，但实际上还是因为太久没有用而导致的不够熟悉。非常费劲的终于调整了好了新 UI，此时已经过去了整整一天了，非常怀念 `autoLayout`。

### context-mune
在正式开始写之前，上文已经说了我一直在翻开源库，主要是不懂得如何下手去，因为距离上一次写 `vue` 已经过去了快两个月了，而且也没搞清楚如何写一个组件，所以中间也有一段时间浪费在了这个上。最后的解决思路让我感到意外：

```vue
<div class="more-menu-wrapper">
    <ul v-show="item.showOption">
        <li>更换分类</li>
        <li>向上移动</li>
        <li>移至顶部</li>
        <li>取消收藏</li>
    </ul>
</div>
```

没想到使用无序列表就可以完成了～在 iOS 中，我会在 `UITableView` 和 `UIStackView` 中纠结。当然只有这样是不行的，当又调整了 UI 后，发现 `...` 和 `context-menu` “融合”在一起了，没有设计图中的“悬浮”效果，最后的解决方法是：

```css
.more-wrapper {
    /* ... */
    position: absolute;
    .more-menu-wrapper {
        position: relative;
        /* ... */
    }
}
```

当继续调整 CSS 时又发现 `context-menu` 的会被其父组件挡住，`context-menu` 的显示范围会限制与其父组件的显示高度，最后得知是 `overflow` 这个属性在最底层的父组件中设置了 `overflow: hidden;`，删除掉，使其为默认的 `visible` 即可显示为 `context-menu` 高度溢出的效果。

### 事件绑定
UI 都调整完后开始绑定事件。因为只是改造 UI，并没有涉及到多少的新逻辑，所以很快的就写出了以下代码：

```vue
<ul v-show="item.showOption">
    <li @click="moveItem(item)">更换分类</li>
    <li @click="moveUp(item)">向上移动</li>
    <li @click="setTop(item)">移至顶部</li>
    <li @click="deleteItem(item)">取消收藏</li>
</ul>
```

`context-menu` 的显示依赖 `v-show`，当页面首次拉取到网络数据时，`data` 中对每个 `listData` 的 `item` 新增了 `context-menu` 显示隐藏的初始化标志位 `item.showOption = false`，且在这四个入口方法中都控制了 `item.showOption` 的改变：

```js
//...
moveUp(item) {
    item.showOption = false;
    // ...
}
//...
```

刷新页面，很愉快的看到了 `context-menu` 的显示，但在点击菜单选项时没有任何反应！一开始以为是标识位的问题，但看来看去没有任何问题哇～本来想去找前端小哥看一眼，但一直不在工位上，最后问了下同组的前端实习生，他认为是 `item.showOption` 字段在数据更新时没有加上，导致后续直接读取时不存在。

但我其实一直纳闷如果 `item.showOption` 字段数据不存在的话，那第一次的页面渲染实际上是有错误的。我们两个人看了一会也没发现具体是哪有问题，最后只能四处寻找前端小哥，没想到他已经被封闭起来做商业化了，我说怎么四处找不到人。

前端小哥在文件中加上了 `debugger` 进行调试，发现进入到 `moveUp` 等一类事件时虽然 `item.showOption` 被修改成功了，一旦出去事件周期外，又被改回去了。

最后发现，问题出在被**冒泡**到了父组件中，调用了 `...` 所绑定的 `onMore` 事件中，而在 `onMore` 事件中 `item.showOption = true`，所以实际上是执行了 `context-menu` 和 `...` 的两者所绑定的事件。解决的方法是：

```vue
<ul v-show="item.showOption">
    <li @click.stop="moveItem(item)">更换分类</li>
    <li @click.stop="moveUp(item)">向上移动</li>
    <li @click.stop="setTop(item)">移至顶部</li>
    <li @click.stop="deleteItem(item)">取消收藏</li>
</ul>
```

使用 `@click.stop` 来阻止冒泡事件。解决完问题后，前端小哥还好奇我做 iOS 怎么会不知道冒泡事件的问题，但实际上在 iOS 中跟前端的思路完全是反过来的。iOS 的事件响应链是逐级传递到子组件中，也就是向下传递，而不是像前端中的向上传递。所以在遇到这个问题时也就完全没有往冒泡的方面去思考。

### 触摸其它区域消失 `context-menu`
在 iOS 中，我会直接封装出一个带有 `UIWindow` 的组件。与 `context-menu` 有关的所有操作与主 `window` 没有任何关系，更别说事件穿透了。所以最终我的做法是多加了一个遮罩层，显示和隐藏的时机与 `context-menu` 的时机保持一致。

最后在我拿着最终的成果去找前端小哥复查时，他对这个做法不满意，还是觉得要使用 `outside-click` 的做法。也就是使用 js 中的事件代理，通过 `e.targe` 去判断。最后找到了可以使用 [`v-outside-click`](https://github.com/ndelvalle/v-click-outside) 进行。`v-outside-click` 有两种引入的方式，为了简洁，我选择了“指令”的方式引入。

在使用 `v-outside-click` 这个库的过程中遇到了一个比较大的问题。`v-outside-click` 此库给我的感觉是用于单个组件，而不适用于多个组件。列表中的每一个 `cell` 都需要带上一个单独的 `context-menu`，如果给每一个 `context-menu` 都绑上一个单独的 `outside-click` 事件，一旦用户的触摸范围不在 `context-menu` 中，则视图上的所有 `context-menu` 都会响应这个 `outside-click` 事件，列表数据一旦多起来，事件响应次数将线性增长。

这个问题跟前端小哥说过后，他说问题不大，那就这样吧～接下来的问题就到了怎么在 `outside-click` 事件中标识出是哪个 `context-menu` 需要隐藏呢？刚开始就按照了以往的套路，直接使用了如下所示的方式：

```html
<div v-click-outside="onClickOutside(item)">
    <!-- ... -->
</div>
```

然后开心看到了报错 `Binding value must be a function or an object`。提示需要传入一个方法？！翻了源码后发现了这么一段：

```js
function processDirectiveArguments(bindingValue) {
  const isFunction = typeof bindingValue === 'function'
  if (!isFunction && typeof bindingValue !== 'object') {
    throw new Error('v-click-outside: Binding value must be a function or an object')
  }

  // ...
}
```

回过头去看之前写的代码，没有问题啊！思来想去还是没弄明白，又去找了前端小哥请求帮忙，经过了一番折腾了，他的结论是这个库应该是有问题的。最后采取的解决方法是：

```html
<div v-click-outside="onClickOutside">
    <p>…</p>
    <!-- 重点 -->
    <div :id="item.metricId" v-show="item.showOption">
        <ul>
            <li>更换分类</li>
            <!-- ... -->
        </ul>
    </div>
</div>
```

```js
onClickOutside (event, el) {
    let queryInstance = el.querySelector('.more-menu-wrapper')         
    if (queryInstance) {
        let metricId = el.querySelector('.more-menu-wrapper').id;
        if (metricId != "") {
            this.listData.some((item) => {
                if (item.metricId == metricId) {
                    item.showOption = false;
                    return true;
                }
            });
        }
    }   
}
```

通过设置 `context-menu` 的 `id` 作为标识，然后在 `v-outside-click` 的指令方法中获取 `id`，通过这个 `id` 去数据源中找到对应的 `item`，从而设置 `item.showOption = false` 来隐藏 `context-menu`。

## 总结
这算是转大前端完成的第一个功能吧，因为不熟悉导致中间出现了一些好玩的事情。客户端和前端的开发流程说大也不大，但要是说没有是绝对不可能的。在一些小的问题上，没有踩过坑或者没有大佬带一带，真的会爬不起来或者就弃坑了，说到底其实还是需要多加学习啊！前端有时候真的挺香的。

