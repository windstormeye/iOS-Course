# CSS
## 基本
### `<ul>` 实现 `<li>` 等分
```html
<ul id="father">
  <li class="children">第一个</li>
  <li class="children">第二个</li>
  <li class="children">第三个</li>
</ul>
```

```css
#father{
  display:flex;
  width:100%;
  height:10rem;
}
.children{
  flex:1;
  text-align:center;
}
```

### 横向布局
1. 使用 `float` 进行浮动；
2. 使用 `inline-block` 行块标签；
3. 使用 `flex` 布局。

### 使用 `div` 还算 `button` 设置按钮呢？
考虑语义化时尽量都按照标准来，但做个性化定制时，需要修改 `button` 的默认属性。考虑兼容性等适配问题时，直接上 `div`，且更容易定制化。

### `float: right` 
当多个 `div` 进行右浮动时，顺序会颠倒，可以让颠倒的 `div` 再嵌套一个容器 `div`，对容器 `div` 设置 `float: right`，对容器内部的 `div` 做 `float: left` 即可。

### `div` 如何设置背景图片
```css
.more-button {
  float: left;
  width: 30px;
  height: 30px;
  background: url(../../../static/images/normal-report/more.png) no-repeat;
  background-size: 30px 30px;
}
```

### 设置 `padding` 再设置 `width: 100%` 后超出父节点宽度
> 在 CSS 盒子模型的默认定义里，你对一个元素所设置的 width 与 height 只会应用到这个元素的内容区。如果这个元素有任何的 border 或 padding ，绘制到屏幕上时的盒子宽度和高度会加上设置的边框和内边距值。——来源 MDN

此时可以使用 `box-sizing` 属性进行调整：
* `content-box`：默认值，如果设置 `width: 100px`，此时再设置 `padding` 或者 `margin` 都会进行叠加；
* `border-box`：设置 `padding` 或者 `margin` 将会包括在 `width` 中，不会进行叠加。

### `let` 和 `var`
在 ES6 之后，使用 `let` 修饰变量在代码块中有效。
