## TranslateP 开发过程

## 选型
使用 SwiftUI，因为 AppKit 实在是太难用了，痛苦。并且作为一个无主界面的 app，运行后只有菜单栏程序，使用[`MenuBarExtra`](https://developer.apple.com/documentation/swiftui/menubarextra)这个 macOS 13 开始支持的 API。

## Accessibility 权限
正在调试运行中的 app 没有加在 Applicatios 文件夹中，需要在 Xcode 的缓存目录下找到你正在运行中的 app，添加到 Accessibility 中。注意用户目录下的 Library 为隐藏目录，使用 shift + command + . 展示 Finder 的隐藏目录。

```bash
/Users/你的用户名/Library/Developer/Xcode/DerivedData 
```