# Layout
这篇文章中将记录 iOS 中 `Layout` 相关的内容。

## `Auto Layout` 介绍
### 有歧义的布局
想要测试一个 `UIVIew/NSView` 的布局约束是否充分，可以在 `loadView` 方法中或者其它建立新视图并添加约束的地方，使用以下属性进行判断。

```swift
view.hasAmbiguousLayout ? "Ambiguous" : "Unambiguous"
```

### 纠正有歧义的布局 & 可视化约束
如果我们想要可视化约束布局，可以使用 macOS 中特有的方法。在 macOS 中，可以通过在任何 `NSWindow` 实例上调用 `visualizeConstraints` 方法，传入需要进行可视化的所以约束集合作为参数即可。

![NSWindow.png](https://i.loli.net/2019/04/24/5cc06c9487d0f.png)

我们可以点击约束，该约束的信息就会显示在 Xcode console 中。

### 内在内容大小
每一个 `UIView` 都一个「内在内容大小」的属性 `view.intrinsicContentSize`，能够直接获取到其内容的真实大小。

> 引发的思考：那这么说做 `UILabel` 的精准伸缩就有救了哈哈哈哈～

### 本章小结
* **问题 1**：一个 54 * 54 点的图像由一个 50 * 50 点的正方形加上一个下拉阴影组成，阴影偏移量为向右 4 点和向下 4 点。将该图像添加到一个图像视图中，并且在两个坐标轴上均相对于其父视图居中时，这个图像中的哪几个几何点位于父视图的中心？写出将 `inset` 赋予该图像的代码。
    - 中心点位于未调整图像的 (27, 27)，已调整图像的 (25, 25)
    - ```swift
        let img = UIImageView(image: UIImage(named: "233"))
        img.alignmentRectInsets = UIEdgeInsets(top: 0, left: 0, bottom: 4, right: 4)
        ```

## 约束