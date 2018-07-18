这篇文章主要记录我在开发PLook的过程中遇到的问题。

1. UIButton的各种状态参考文章：[https://www.jianshu.com/p/57b2c41448bf](https://www.jianshu.com/p/57b2c41448bf)

2. **照片编辑**。重写`scrollView`的`hitTest`方法。当用户手指发生触摸时，前150ms会被`scrollView`截获，当150ms后用户的手指还在原位的话，`scrollView`则自动下发给对应的可以进行处理的控件，需要做的事情就是重写`scrollView`的`hitTest`方法，`isKindOf`用户想要触摸上的控件类（[参考文章](https://www.jianshu.com/p/2bb30c3d2408)）
```swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    let view = super.hitTest(point, with: event)
    if (view?.isKind(of: PJEditImageTouchView.self))! {
        self.isScrollEnabled = false
    } else {
        self.isScrollEnabled = true
    }
    return view
}
```

3. 记录一下。又忘了`UIImageView.userInterfaceEnable`默认是`NO`

4. `viewDidLoad`和`loadView`区别。`loadView`创建`ViewController`一个空白的`view`，默认会创建一个空白的`view`，`viewDidLoad`是通过xib加载视图后的调用方法，不过最后都会去调用`viewDidLoad`，所以之前一直就在`viewDidLoad`方法中初始化视图。[参考文章1](https://www.cnblogs.com/mjios/archive/2013/02/26/2933667.html)和[参考文章2](https://github.com/bestswifter/blog/blob/master/articles/uiview-life-time.md)

5. 遇到一个非常奇怪的问题，`Swift上`触发了通知、闭包、代理后，OC接受到执行pushVC，特别特别特慢，不知道为啥的。😳。

**解决：**点击拍照的时候报了一个错：
`This application is modifying the autolayout engine from a background thread after the engine was accessed from the main thread. This can lead to engine corruption and weird crashes.`

用回到主线程的方法后重新调用`viewDelegate`回调代理方法，搞定！因此猜测，`captureOutput didOutput`是在子线程的跑的方法

`videoDataOutPut.setSampleBufferDelegate(self, queue: .global())`是这个方法导致的，给output流传入的是一个正常等级全局子线程，故应该合并到主线程中再进行各种操作。如果不想这么做，那就直接丢入主线程`videoDataOutPut.setSampleBufferDelegate(self, queue: DispatchQueue.main)`

6. `NSTimer`和`CADisplayLink`做定时任务时，默认加到的`runloop`是`NSRunLoopDefaultMode`，当视图滑动时，`runloop`只会处理`UITrackingRunLoopMode`，并不会处理`timer`和`CADisplayLink`，可以把`runloop`的`mode`改为`NSRunLoopCommonMode`s即可。[参考文章](https://blog.csdn.net/wzzvictory/article/details/22417181)