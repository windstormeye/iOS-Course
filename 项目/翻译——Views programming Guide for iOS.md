这是我第一个翻译的英文官方文档——View Programming Guide for iOS，原本是想加强自己对iOS开发中视图方面的理解，从官方文档先入手，但是突然想到自己还要考四六级等等其它因素，遂有了把自己阅读的原版官方文档翻译成中文的冲突，提升下自己的英语能力。

欢迎PR，一起完成！原文地址：[https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html](https://developer.apple.com/library/content/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/Introduction/Introduction.html)

## View Programming Guide for iOS——iOS中的视图编程指导

在iOS中，你（觉得用“我们”会更好）会使用多个`window`和多个`View`在屏幕上展示你的应用程序内容。`window`本身不会包涵任何的可见内容，但是会提供一个基础容器给我们应用程序中的`View`使用。`View`定义了一部分你想要填入`window`中的内容。举个例子，你可以有用于显示图片、文本、形状或者以上内容的组合`view`，甚至你还可以使用`View`去组织和管理其它`View`。

### 扫一眼（😒At a Glance真这么翻译嘛？）

每一个应用程序至少拥有一个`window`和`view`去展示它的内容。`UIKit`框架和其它系统框架预先定义了`view`供你使用去展示你的内容。这些`view`涵盖了简单的`button`和文本`Lable`到复杂的`view`，比如`table view`、`picker view`和`scroll view`。在这些预先定义好的`view`中没有提供你想要的`view` ，你可以自定义`view`和绘制并自己管理事件。

#### `views`管理你的应用程序可视内容

一个`view`是`UIView`类（或者它的子类）的实例并且管理了一个你的应用程序`window`中的矩形区域。`Views`能够响应绘制的内容，处理多点触控事件和管理该`Views`中的子视图约束。在一个矩形`View`区域中涉及到绘图可以使用图像技术，比如`Core Graphics`、`OpenGL ES`或者`UIKit`去绘制形状、图片和文本。`View`能够直接的响应手势识别或触摸事件。在`view`的等级制度中，父视图是其子视图位置和大小的主管，并且还能够做很多的动态事情。父视图的这个能力能够动态的让你的`views`去适应改变条件，比如说界面旋转和动画。

你可以认为`views`是构建你的用户界面中的一个砖块。你会经常的使使用一系列的`views`去构建一个等级视图去展示你的内容，而不是只使用一个`view`去展示。每一个在等级视图中的`view`都表现了用户界面中的一部分，并且通常是用于优化一个具体的内容类型，例如，`UIKit`有很多个用于优化图像、文本和其它类型内容的`view`。


#### 相关内容。。。。（因为我还没翻译到后边，就不贴了，免得又是英文，写完再贴）


`Windows`配合显示你的`views`
一个window是`UIWindow`类的实例并且能够处理全部呈现于你的应用程序用户界面上的内容。`window`与`view`共同工作（并且还是视图控制器的拥有者），去管理和改变相互之间的等级视图。在大部分的时候，你的应用程序`window`并不会发生改变。`window`创建之后，它不会发生改变，只有通过它显示的`views`才会发生改变。每一个应用程序都只至少有一个`window`在设备的主屏幕上去显示这个应用程序用户界面。如果有外部显示设备连接到了这个设备，应用程序能够较好的在这个外部显示设备上创建第二个window去呈现内容（这句话翻译不确定）。

#### 动画提供了用户可见的界面改变反馈

动画提供了关于你的层级视图用户可见反馈的改变。系统定义了呈现模态视图（或单一视图）和位于不同组合视图之间过渡的动画。不过，`view`有很多能够直接修改动画的属性。例如，通过动画你可以改变`view`的透明度，在屏幕中的位置，大小，背景颜色或者其它的属性。如果你直接使用底层的`Core Animation`层对象进行工作，你能够作出其它很多不错的动画。

#### `interface Builder`的扮演的角色

`interface Builder`是一个图形构造和配置你的应用程序`windows`和`view`的工具。使用`interface Builder`，你能够集中你的`views`并处置它们的在`nib`文件中的位置，它是一个存储了你可以自由移动版本的`view`或其它对象的资源文件，当你在`runtime`加载一个`nib`文件时，你可以使用你的代码去操纵这些位于真实对象中的`view`。

你不得不去进行创建你的应用程序界面工作时，使用`interface Builder`会变得非常简单。因为iOS始终支持并合并进了`interface Builder`和`nib`文件，只需要一点点的时间就可以把你应用程序的设计工作合并到`nib`文件中。

关于更多的如何使用`interface Builder`信息，请看Interface Builder User Guide。关于如何使用`view controller`管理包涵这些view`的`nib`文件，请看[View Controller Programming Guide for iOS](https://developer.apple.com/library/content/featuredarticles/ViewControllerPGforiPhoneOS/index.html#//apple_ref/doc/uid/TP40007457)中的创建自定义视图控制器。
