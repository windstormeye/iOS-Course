# Layout
这篇文章中将记录 iOS 中 `Layout` 相关的内容。

## `Auto Layout`
### 有歧义的布局
想要测试一个 `UIVIew/NSView` 的布局约束是否充分，可以在 `loadView` 方法中或者其它建立新视图并添加约束的地方，使用以下属性进行判断。

```swift
view.hasAmbiguousLayout ? "Ambiguous" : "Unambiguous"
```

### 纠正有歧义的布局 & 可视化约束
如果我们想要可视化约束布局，可以使用 macOS 中特有的方法。在 macOS 中，可以通过在任何 `NSWindow` 实例上调用 `visualizeConstraints` 方法，传入需要进行可视化的所以约束集合作为参数即可。

![NSWindow.png](https://i.loli.net/2019/04/24/5cc06c9487d0f.png)

我们可以点击约束，该约束的信息就会显示在 Xcode console 中。

