# Xcode

##  `liveView`
在Xcode 11 中推出了针对 `SwiftUI` 的 `liveView` 功能。前提是需要 macOS 10.15 及 Xcode 11。

在安装好以上两个必须的条件后，此时如果出现了 `Failed to code sign ContentView.swift` 的提示，说明还没有针对 Xcode 11 下载 `command line tools`，手动执行 `xcode-select --install`。

如果还是不行，原因很有可能是因为更新了系统，需要重新下载一遍 `command line tools`，下载完成后，使用命令 `xcode-select --s` 指定工作目录
