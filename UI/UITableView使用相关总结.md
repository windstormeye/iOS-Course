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

// 持续更新中......
