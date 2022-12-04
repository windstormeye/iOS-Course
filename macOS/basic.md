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