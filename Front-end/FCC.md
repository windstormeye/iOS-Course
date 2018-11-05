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