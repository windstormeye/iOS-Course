## 小功能实现细节

## 悬浮窗口
类似网易云桌面悬浮单词，本质上都是基于画中画做的，第一种方案是把 view 转视频，按时塞帧，第二种方案是直接获取 pip window 贴自定义 view。

可见：[https://github.com/CaiWanFeng/PiP](https://github.com/CaiWanFeng/PiP)

## 防录屏
系统提供的截图和录屏的通知都是结束之后的。

最佳方案是 DRM，但本地 DRM 太复杂了，而且成本比较大。

目前看了几个投机取巧的方案都是基于 UITextField 开启 isSecureTextEntry 后拿私有视图 _UITextFieldCanvasView 做的，太 trick 了，还在找有没有牛逼的方案

基于 _UITextFieldCanvasView 可见：https://blog.wyan.vip/2024/02/iOS_avoid_screen_capture.html

但系统又提供了 `UIScreen.capturedDidChangeNotification` 这个通知，可以直接拿到是否在录屏时机，且从 iOS 11 开始支持。

```swift
NotificationCenter.default.addObserver(self, selector: #selector(screenCaptureChanged), name: UIScreen.capturedDidChangeNotification, object: nil)

@objc func screenCaptureChanged(notifi: Notification) {
    if let screen = view.window?.windowScene?.screen, screen.isCaptured {
        captureView.isHidden = false
    } else {
        captureView.isHidden = true
    }
}
```


## 防截图
方案一：监听 UIApplicationUserDidTakeScreenshotNotification 通知，然后去删相册里最新的一张照片，需要相册访问权限。这方案肯定不行。