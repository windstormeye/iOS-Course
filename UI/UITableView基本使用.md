本文主要总结在日常撸码中，遇到`UITableView`的各种需求demo及各种问题和骚操作集合。持续更新中......

## 差异

### Android

如果你有做过Android，那么`UITableView`就是`ListView`，`ListView`中的`item`对应的就是`UITableView`的`UITableViewCell`，相同点是它们两者都有重用机制，都可供用户进行滑动操作进行数据的展示，而关于不同点从基础的父类等其它方面来看，差异性大于相同性，其中我觉得最大的差别就是显示风格上，`UITableView`有`Group`和`Plain`样式，而`ListView`只有一个`Plain`样式，这两种的模式的区别大家可以打开微信，微信的`微信`和`通讯录`两个tabVC就是`UITableView`的两个风格展示。


### React-Native

React-Native中并没有iOS和Android中传统意义的上的`UITableView`或`ListView`，但真的有`ListView`这个组件，因为是支持`flex`布局，因此在RN中的这个`ListView`可以认为是iOS中的`UITableView`和`UICollectionView`的融合，Android中的`ListView`和`GridView/Layout`（或者其它布局方式）的融合。

关于React-Native的相关内容介绍，可以参考我的[这一系列文章](../项目/React-Native记〇.md)

### 微信小程序

微信小程序中也没有单独的一个控件做列表展示，而用`<View>`组件堆积起来后自动延伸成一个列表，而且同样也是支持`flex`布局，因此其实际上也是iOS中的`UITableView`和`UICollectionView`的融合，Android中的`ListView`和`GridView/Layout`的融合。

关于微信小程序的相关内容介绍，可以参考我的[这一系列文章](../项目/小程序初探.md)


## 搭建

在iOS开发中搭建`UITableView`，可以通过的`StoryBoard / Xib`和纯代码的方式进行，一般来说，并不推荐直接使用`StoryBoard / Xib`进行界面搭建，但这有一个的例外，如果你的页面是静态的，比如个人中心，类似微信的`我`，基本上除了每次的版本更新外，没见过动态更新的时候（当然，有可能也会有）。

类似这种页面，我们都可以使用`StoryBoard / Xib`来搭建，而且会更快更简洁明了，但是如果我们的页面类似微博首页、朋友圈等的信息流动态更新十分频繁，那么尽可能的使用纯代码方式进行布局，因为你无法保证列表的每个单元格是否都一致，显示的内容同样是否也一致，其实最致命的，如果我们不用纯代码布局，很难保证后续接手的同学是否能够快速上手撸码。😂。

### 使用`StoryBoard / Xib`搭建

使用`StoryBoard / Xib`进行`UITableView`的搭建有两种方式，一种是在已有的`ViewController`上拖拽进`UITableView`；第二种是直接在SB中拖拽一个`UITableViewController`。简单使用方法如下图所示：

<img src="https://i.loli.net/2018/04/20/5ad964a295f0d.png" width = "100%" height = "100%" align=center />

<img src="https://i.loli.net/2018/04/20/5ad964a2a3ffa.png" width = "100%" height = "100%" align=center />

那这两张方式的区别在哪呢？先抛开`UITableView`，就拿微信朋友圈的例子来说，如果是我自己去做一个微信朋友圈，最底层我会拿一个`UIViewController`，然后用纯代码的方式搭建出一个`UITableView`，用`Xib`搭建出每一个单元格`Cell`，每条朋友圈下的评论用纯代码的方式去动态计算cell的行高，当然这其中肯定涉及到了很多其它内容。

这其中有个需要注意的地方，每个cell使用`Xib`去做，为啥cell要通过`Xib`去做呢？首先，如果你的页面比较复杂，比如去年年初时我实习做的其中一个App：

<img src="https://i.loli.net/2018/04/20/5ad9679e5c299.png" width = "40%" height = "40%" align=center />

如上图所示的页面，如果是拿纯代码布局，真的，我说实话，非常有可能一上午的时间都都搭在上边了，但是如果拿`StoryBoard / Xib`搭建，很有可能半小时不到就好了。

通过以上的说法，大家应该能有对纯代码和`StoryBoard / Xib`搭建页面有了一个基本的认识，各自的优缺点也有了浅显的认识。接下来，我们将通过结合几个小案例疏通`UITableView`的基本使用。



// 持续更新中......
