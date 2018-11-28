# FCC 学习笔记
这是我在 [freecodecamp.one](http://freecodecamp.one) 上学习遇到的问题以及值得记录的点。

## `<main>` 标签
该标签用与在 `<body>` 标签内标识出“内容主体”，因为同时还会有 `<nav>、<header>、<footer>` 等标签，把页面主体内容写在 `<main>` 标签中，有这样搜索引擎进行检索，它只应包含与页面中心主题相关的信息，而不应包含如导航连接、网页横幅等可以在多个页面中重复出现的内容。

`main` 标签的语义化特性可以使辅助技术快速定位到页面的主体。有些页面中有 “跳转到主要内容” 的链接，使用 `main` 标签可以使辅助设备自动获得这个功能。例子如下：

```html
<!DOCTYPE html>  
<html lang="en">  
<head>  
<meta charset="UTF-8" />  
<title>Using main</title>  
</head>  
<body>  
    <header>My page</header>  
    <nav>  
        [url=#]Home[/url]  
    </nav>  
    <main>  
        <article>  
            <h1>My article</h1>  
            Content  
  
        </article>  
        <aside>  
            More information  
  
        </aside>  
    </main>  
    <footer>Copyright 2013</footer>  
</body>  
</html>  
```

## HTML 5 `<a>` 标签的 `target` 属性
target 属性规定在何处打开被链接文档。只能在 href 属性存在时使用。

`<a href="http://www.w3school.com.cn" target="_blank">Visit W3School</a>`


值 | 描述
---- | ----
_blank | 在新窗口中打开被链接文档。
_self | 在被点击时的同一框架中打开被链接文档（默认）。
_parent | 在父框架中打开被链接文档。
_top | 在窗口主体中打开被链接文档。


## `<br>`
`<br>` 标签是空标签（意味着它没有结束标签，因此这是错误的：`<br></br>`）。在 XHTML 中，把结束标签放在开始标签中，也就是 `<br />`。

## HTML `<meta>` 标签
`<meta>` 元素可提供有关页面的元信息（meta-information），比如针对搜索引擎和更新频度的描述和关键词。

## `class` 选择器
在 `<style></style>` 区域，这么写：

```html
<style>
    .red-text {
        color: red;
    }
</style>
```

在 `<body></body>` 区域，这么写：

```html
<body>
    <h2 class="red-text"> 2333 </h2>
</body>
```
### 其它
在 `<style>` 标签里面声明的 `class` 顺序十分重要。第二个声明始终优于第一个声明。因为 `.blue-text` 在 `.pink-text` 的后面声明，所以 `.blue-text` 会覆盖 `.pink-text` 的样式。

## 更换字体
在 `<style>` 标签之前（或者顶部），写下：

```html
<link href="https://fonts.googleapis.com/css?family=Lobster" rel="stylesheet" type="text/css">
```

可以去 [Google 免费字体库](https://fonts.google.com/) 中找一些自己需要的字体。

## 字体降级
所有浏览器都有几种默认的字体，包括但不限于：`monospace`, `serif`, `sans-serif`, `Helvetica` 等。

当我们引用外部字体却因为某些原因导致字体加载失败时，可以采用“字体降级”：

```css
p {
    font-family: Lobster, sans-serif;
}
```

当找不到 `Lobster` 字体时，将自动降级为 `sans-serif`，但如果设备中已经安装了 `Lobster` 字体，则不会降级，因为浏览器会找到这个字体（目前暂不知道浏览器怎么寻找字体）。

## CSS 设置圆形
```css
.image {
    border-radius: 50%;
}
```

## id 属性（id 选择器）
在 CSS 里，`id` 不可以重用，只应用于一个元素上且`id` 的优先级要高于 `class`，如果一个元素同时应用了 `class` 和 `id`，并设置样式有冲突，会优先应用`id` 的样式。

```css
#cat-photo-element {
  background-color: green;
}
```

## padding（内边距），margin（外边距）和border（边框）
* `padding` 控制着元素内容与border之间的空隙大小。
* `margin`（外边距）控制元素的边框与其他元素之间的距离。

## 属性选择器
```css
[type='checkbox'] {
    margin: 10px 0 15px 0px
}
```
这样就把 `type` 类型为 `checkbox` 的复选框一同进行设置了。
```html
<form action="/submit-cat-photo" id="cat-photo-form">
    <label><input type="radio" name="indoor-outdoor">室内</label>
    <label><input type="radio" name="indoor-outdoor">室外</label><br>
    <label><input type="checkbox" name="personality">忠诚</label>
    <label><input type="checkbox" name="personality">懒惰</label>
    <label><input type="checkbox" name="personality">积极</label><br>
    <input type="text" placeholder="猫咪图片地址" required>
    <button type="submit">提交</button>
</form>
```

## 内联样式的优先级高于 id 选择器

## !important
在很多时候，你使用 CSS 库，有时候它们声明的样式会意外的覆盖你的 CSS 样式。当你需要保证你的 CSS 样式不受影响，你可以使用 `!important`:

```css
.pink-text {
    color: pink !important; 
}
```

## 绝对单位和相对单位
单位长度的类型可以分成 2 种，一种是相对的，一种是绝对的。例如，`in` 和 `mm` 分别代表着英寸和毫米。绝对长度单位会接近屏幕上的实际测量值，不过不同屏幕的分辨率会存在差异，可能会导致一些误差。

相对单位长度，就像 `em` 和 `rem`，它们会依赖其他长度的值。就好像 `em` 的大小基于元素的字体的 `font-size` 值，如果你使用 `em` 单位来设置 `font-size` 值，它的值会跟随父元素的 `font-size` 值来改变。

## 每一个 HTML 页面都含有 body 元素

## 定义并使用自己的 CSS 变量
### 定义
创建一个 CSS 变量，你只需要在变量名前添加两个破折号，并为其赋值，例子如下：
```css
--penguin-skin: gray;
```
这样会创建一个 `--penguin-skin` 变量并赋值为 `gray`（灰色）。

### 使用
创建变量后，CSS 属性可以通过引用变量名来使用它的值。

```css
background: var(--penguin-skin);
```

### 其它
使用变量来作为 CSS 属性值的时候，可以设置一个备用值来防止由于某些原因导致变量不生效的情况。

或许有些人正在使用着不支持 CSS 变量的旧浏览器，又或者，设备不支持你设置的变量值。为了防止这种情况出现，那么你可以这样写：

`background: var(--penguin-skin, black);`
这样，当你的变量有问题的时候，它会设置你的背景颜色为黑色。

##  `:root` 变量
就像 `html` 是 `body` 的容器一样，你也可以认为 `:root` 元素是你整个 `HTML` 文档的容器。在 `:root` 创建的变量，在整个网页里面都能生效。

```css
<style>
  :root {
    
    /* add code below */
    --penguin-belly: pink;
    /* add code above */
  }
</style>
```

当你在:root里创建变量时，这些变量的作用域是整个页面。如果在元素里创建相同的变量，会重写:root变量设置的值。

## 使用媒体查询更改变量
CSS 变量可以简化媒体查询的方式。例如，当屏幕小于或大于媒体查询所设置的值，通过改变变量的值，那么应用了变量的元素样式都会得到响应修改:

```css
@media (max-width: 350px) {
    :root {
        
        /* add code below */
        --penguin-size: 200px;
        --penguin-skin: black;
        /* add code above */
    }
}
```

## `text-align`
* `text-align: justify;` 可以让除最后一行之外的文字两端对齐，即每行的左右两端都紧贴行的边缘。

* `text-align: center;` 可以让文本居中对齐。

* `text-align: right;` 可以让文本右对齐。

* `text-align: left;` 是 `text-align` 的默认值，它可以让文本左对齐。

## `strong` 标签加粗文本
你可以使用 `strong` 标签来加粗文字。添加了 `strong` 标签后，浏览器会自动给元素应用 `font-weight:bold;` 。

## `u` 标签
你可以使用u标签来给文字添加下划线。添加了 `u` 标签后，浏览器会自动给元素应用 `text-decoration: underline;` 。

## `em` 标签
你可以使用 `em` 标签来强调文本。由于浏览器会自动给元素应用 `font-style: italic;`，所以文本会显示为斜体。

## `s` 标签
你可以用 `s` 标签来给文字添加删除线，它代表着这段文字不再有效。添加了 `s` 标签后，浏览器会自动给元素应用`text-decoration: line-through;`。

## `hr` 标签
你可以用 `hr` 标签来创建一条宽度撑满父元素的水平线。它一般用来表示文档主题的改变，在视觉上将文档分隔成几个部分。在 `HTML` 里，`hr` 是自关闭标签，所以不需要一个单独的关闭标签。

## `text-transform` 给文本添加大小写效果
`CSS` 里面的 `text-transform` 属性来改变英文中字母的大小写。它通常用来统一页面里英文的显示，且无需直接改变 `HTML` 元素中的文本。下面的表格展示了 `text-transform` 的不同值对文字 “Transform me” 的影响。

Value | Result
--- | ---
lowercase | "transform me"
uppercase | "TRANSFORM ME"
capitalize | "Transform Me"
initial | 使用默认值
inherit | 使用父元素的text-transform值。
none | Default:不改变文字。

## `font-weight`
`font-weight` 属性用于设置文本中所用的字体的粗细。

## `line-height`
CSS 提供 `line-height` 属性来设置行间的距离。行高，顾名思义，用来设置每行文字所占据的垂直空间。

## 伪类
比如，超链接可以使用 `:hover` **伪类选择器**定义它的悬停状态样式。下面是悬停超链接时改变超链接颜色的 CSS：
```css
a:hover {
  color: red;
}
```

## 更改元素的相对位置——`position`
在 CSS 里一切 HTML 元素皆为盒子，也就是通常所说的 **盒模型**。块级元素自动从新的一行开始（比如标题、段落以及 div），行内元素（也称：内联元素）排列在上一个元素后（比如图片以及 span）。元素默认按照这种方式布局称为文档的**普通流**，同时 CSS 提供了 `position` 属性来覆盖它。

```css
p {
  position: relative;
  bottom: 10px;
}
```

## 更改元素的绝对位置——`absolute`
 CSS `position` 属性的取值选项 `absolute`，`absolute` 相对于其包含块定位。和 `relative` 定位不一样，`absolute` 定位会将元素从当前的文档流里面移除，周围的元素会忽略它。可以用 CSS 的 `top`、`bottom`、`left `和 `right` 属性来调整元素的位置。

`absolute` 定位比较特殊的一点是元素的定位参照于最近的已定位祖先元素。如果它的父元素没有添加定位规则（默认是`position:relative;`）,浏览器会继续寻找直到默认的 `body` 标签。

## `fixed` 定位
`fixed` 定位，它是一种特殊的绝对（absolute）定位，区别是其包含块是浏览器窗口。和绝对定位类似，`fixed` 定位使用 `top`、`bottom`、`left` 和 `right` 属性来调整元素的位置，并且会将元素从当前的文档流里面移除，其它元素会忽略它的存在。

`fixed` 定位和 `absolute` 定位的最明显的区别是 `fixed` 定位元素不会随着屏幕滚动而移动。

## 元素浮动
浮动元素不在文档流中，它向左或向右浮动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。通常需要用 `width` 属性来指定浮动元素占据的水平空间。

## `z-index` 属性更改重叠元素的位置
在 HTML 里后出现的元素会默认显示在更早出现的元素的上面。你可以使用 `z-index` 属性指定元素的堆叠次序。`z-index` 的取值是整数，数值大的元素优先于数值小的元素显示。

## 让元素水平居中
一种常见的实现方式是把块级元素的 `margin` 值设置为 `auto`。同样的，这个方法也对图片奏效。图片默认是**内联元素**，但是可以通过设置其 `display` 属性为 `block` 来把它变成**块级元素**。

## 线性渐变器
`background: linear-gradient(45deg, #CCFFFF, #FFCCCC);`

## 重复线性渐变器
```css
background: repeating-linear-gradient(
    45deg,
    yellow 0px,
    yellow 40px,
    black 40px,
    black 80px
);
```
## 通过添加细微图案作为背景图像来创建纹理
终于明白了公司内网上标记名字的背景图像是怎么做的了

`background: url(https://i.imgur.com/MJAkxbh.png);`

## 更改元素大小
`transform: scale(1.5);`

## 鼠标悬停时缩放元素
```css
div:hover {
    transform: scale(1.1);
}
```

## 倾斜
```css
    transform: skewX(24deg);
    transform: skewY(-10deg);
```

## 画一个月亮 🌛
`blur-radius` => 模糊半径，`spread-radius` => 辐射半径，`transparent` => 透明的，`border-radius` => 圆角边框
```css
background-color: transparent;
border-radius: 50%;
box-shadow: 25px 10px 0px 0px green;
```

## 画一个爱心 ❤️
```css
<style>
<!-- 先画一个正方形 -->
.heart {
  position: absolute;
  margin: auto;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: pink;
  height: 50px;
  width: 50px;
  transform: rotate(-45deg);
}
<!-- 再画一个圆形在左上角 -->
.heart:after {
  background-color: pink;
  content: "";
  border-radius: 50%;
  position: absolute;
  width: 50px;
  height: 50px;
  top: 0px;
  left: 25px;
}
<!-- 再画一个圆形右上角 -->
.heart:before {
  content: "";
  background-color: pink;
  border-radius: 50%;
  position: absolute;
  width: 50px;
  height: 50px;
  top: -25px;
  left: 0px;
}
</style>
<div class = "heart"></div>
```

## 动画
```css
#anim {
  animation-name: colorful;
  animation-duration: 3s;
}
@keyframes colorful {
  0% {
    background-color: blue;
  }
  100% {
    background-color: yellow;
  }
}
```

## 修改动画的填充模式
动画在持续时间之后重置了，所以按钮又变成了之前的颜色。如果我们想要的效果是按钮在悬停时始终高亮。

`animation-fill-mode: forwards;`

## 动画次数
`animation-iteration-count: infinite;`

## 使用关键字更改动画定时器
`animation-timing-function: ease-out;`

## 当图片无法读取或需要提供描述信息时
`<img src="samuraiSwords.jpeg" alt="这里填写图片描述信息">`

## 每个页面应该只有一个h1标签，用来说明页面主要内容。h1标签和其他的标题标签可以让搜索引擎获取页面的大纲

## 音频播放
```css
<audio controls>
    <source src="https://s3.amazonaws.com/freecodecamp/screen-reader.mp3" type="audio/mpeg" />
</audio>
```

## `label` 标签的 `for` 属性
`label` 元素不会向用户呈现任何特殊效果。不过，它为鼠标用户改进了可用性。如果您在 `label` 元素内点击文本，就会触发此控件。就是说，当用户选择该标签时，浏览器就会自动将焦点转到和标签相关的表单控件上。前提得跟表单内的输入组件 id 相同，例如：
```html
<form>
  <label for="name">Name:</label>
  <input type="text" id="name" name="name">
</form>
```

## `<sub>` 标签
`<sup>` 标签可定义上标文本。

包含在 `<sup>` 标签和其结束标签 `</sup>` 中的内容将会以当前文本流中字符高度的一半来显示，但是与当前文本流中文字的字体和字号都是一样的。

## 将键盘焦点添加到元素中
```html
<p tabindex = "0">Instructions: Fill in ALL your information then click <b>Submit</b></p>
```

这样就会当用户使用 `tab` 键进行切换时，会把键盘焦点聚集在该标签上。

# 响应式 Web 设计原则
## 创建一个媒介查询
设备的高度小于或等于 `800p`x 时，`p` 标签的 `font-size` 为 `12px` 的媒体查询：
```css
@media (min-height: 800px) {
    p {
      font-size: 12px;
    }
}
@media (max-height: 800px) {
    p {
        font-size: 12px;
    }
}
```

## 使排版根据设备尺寸自如响应
视窗单位相对于设备的视窗尺寸 (宽度或高度) ，百分比是相对于父级元素的大小。

四个不同的视窗单位分别是：

* vw：如 10vw 的意思是视窗宽度的 10%。
* vh： 如 3vh 的意思是视窗高度的 3%。
* vmin： 如 70vmin 的意思是视窗中较小尺寸的 70% (高度 VS 宽度)。
* vmax： 如 100vmax 的意思是视窗中较大尺寸的 100% (高度 VS 宽度)。