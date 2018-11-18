# FCC 学习笔记
这是我在 [freecodecamp.one](http://freecodecamp.one) 上学习遇到的问题以及值得记录的点。

## `<main>` 标签
该标签用与在 `<body>` 标签内标识出“内容主体”，因为同时还会有 `<nav>、<header>、<footer>` 等标签，把页面主体内容写在 `<main>` 标签中，有这样搜索引擎进行检索，例子如下：

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