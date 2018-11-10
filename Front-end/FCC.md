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

## HTML 5 <a> 标签的 target 属性
target 属性规定在何处打开被链接文档。只能在 href 属性存在时使用。

`<a href="http://www.w3school.com.cn" target="_blank">Visit W3School</a>`

值 | 描述
---- | ----
_blank | 在新窗口中打开被链接文档。
_self | 在被点击时的同一框架中打开被链接文档（默认）。
_parent | 在父框架中打开被链接文档。
_top | 在窗口主体中打开被链接文档。

## <br>
`<br>` 标签是空标签（意味着它没有结束标签，因此这是错误的：`<br></br>`）。在 XHTML 中，把结束标签放在开始标签中，也就是 `<br />`。

## HTML <meta> 标签
`<meta>` 元素可提供有关页面的元信息（meta-information），比如针对搜索引擎和更新频度的描述和关键词。
