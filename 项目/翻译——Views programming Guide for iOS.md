这是我第一个翻译的英文官方文档——View Programming Guide for iOS，原本是想加强自己对iOS开发中视图方面的理解，从官方文档先入手，但是突然想到自己还要考四六级等等其它因素，遂有了把自己阅读的原版官方文档翻译成中文的冲突，提升下自己的英语能力。

欢迎PR，一起完成！原文地址：[https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html)

## View Programming Guide for iOS——iOS中的视图编程指导

在iOS中，你（觉得用“我们”会更好）会使用多个`window`和多个`View`在屏幕上展示你的应用程序内容。`window`本身不会包涵任何的可见内容，但是会提供一个基础容器给我们应用程序中的`View`使用。`View`定义了一部分你想要填入`window`中的内容。举个例子，你可以有用于显示图片、文本、形状或者以上内容的组合`view`，甚至你还可以使用`View`去组织和管理其它`View`。

### 扫一眼（😒At a Glance真这么翻译嘛？）

每一个应用程序至少拥有一个`window`和`view`去展示它的内容。`UIKit`框架和其它系统框架预先定义了`view`供你使用去展示你的内容。这些`view`涵盖了简单的`button`和文本`Lable`到复杂的`view`，比如`table view`、`picker view`和`scroll view`。在这些预先定义好的`view`中没有提供你想要的`view` ，你可以自定义`view`和绘制并自己管理事件。

#### `views`管理你的应用程序可视内容

一个`view`是`UIView`类（或者它的子类）的实例并且管理了一个你的应用程序`window`中的矩形区域。`Views`能够响应绘制的内容，处理多点触控事件和管理该`Views`中的子视图约束。
