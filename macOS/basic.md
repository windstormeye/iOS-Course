## 基础知识

### 窗体
macOS 的窗体大小由内容决定，只需要限制窗体其中内容最小 size 即可。

### 如何判断是否能引入一个库/平台类型

```swift
#if canImport(UIKit)
import UIKit
public typealias PlatformImageType = UIImage
#else
import Cocoa
public typealias PlatformImageType = NSImage
#endif
```

### dmg 软件分发
* 本质利用了 `.DS_Store` 文件记录了文件和文件夹的位置信息，可以做出精美的图标展示。
* 参考文章：[再谈 .DS_Store：兼论 Windows 与 macOS Finder 的布局理念差异](https://sspai.com/prime/story/on-dsstore)