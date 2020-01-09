# Basic HTML and HTML5

* 作为惯例，所有的 HTML 标签都是小写，比如是 <p></p>，而不是<P></P>
* 所有的 img 标签都必须有 alt 修饰。该标签可被用户读屏和如果图片加载不出来时进行文本展示。
* 使用 a 标签通过 href 进行跳转时，增加 target="_blank" 可以在新的标签页打开。
* 一般来说我们在 <a> 标签中通过 href 进行跳转，但如果在 href 中使用 #about 等类似的标识符，这将会挑战到当前页面对于的 id 的标签上，如果此时只给了 #，则只会刷新当前页面。
* <input> 如果为必填框，可以设置 required 标记
* 每一个 <input> type 为 radio 的标签都应该内嵌在各自独立的 <label> 标签中。
* <label> 标签作用在 <input> 类型标签的作用在于 accessibility，供读屏软件定位到输入区域时给用户提示


# Basic CSS
* CSS 下的 font-family 设置字体时，先在编辑器最顶部导入字体源，然后再写入对应的字体名字，若当前字体不可用时，可以多写一个浏览器自带字体，做个捞底。
* Id 修饰有助于JS 调用，而且应该全局唯一。浏览器虽然不会强制要求，但这是一个最佳实践。
* Id 不可被服用，且应该只对一个元素生效。当 id 和 class 所挂载的 css 发生冲突时，id 的优先级大于 class。
* 类似 <input> 标签给了 type 后，想要针对不同的 type 做 CSS，可以通过这种方式
[type = ‘checkbox’] = {
		
}
* Class 中修饰的内容大于 body 中修饰的内容，后边更新的属性会覆盖前一个属性
* 写在同一个元素中的不同 class，第二个 class 修饰的内容会复写第一个修饰的 class 中修饰的内容
* 不管 class 修饰了多少次，id 再去修饰同一个属性的优先级是最高的
* 当比较难重写 CSS 时，可以使用 !important 进行强制使用
* 十六进制颜色值是按照 #RRGGBB 的格式进行排布，0 为最低色值，F 为最高。
* 如果涉及到一个 CSS 属性很多地方都有用到，可以使用 CSS 变量进行统一处理：

```css
  .penguin {
    --penguin-skin: black;
  }

  .penguin-top {
    background: var(--pengiun-skin);
  }
```

* CSS 变量有些浏览器并不支持，为了提升可用性，可以重复定义一个相同的 CSS 属性。

```css
.red-box {
    background: red;
    background: var(--red-color);
  }
```

* CSS 变量可以被重写
* 媒体查询

```css
  :root {
    --penguin-size: 300px;
    --penguin-skin: gray;
    --penguin-belly: white;
    --penguin-beak: orange;
  }

  @media (max-width: 350px) {
    :root {
      --penguin-skin: black;
      --penguin-size: 200px;
    }
  }
```


* CSS :root 伪类选择器？


# Applied Visual Design
* text-align: justify;
    * 文字向两侧对齐，对最后一行无效。
* text-align: justify-all;
* 同上，但强制最后一行两端对齐。
* <strong> 标签可以替代 CSS 属性 font-weight: bold; 作用。
* <u> 标签加下划线
* <em> 加斜体







