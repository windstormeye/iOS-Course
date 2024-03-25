## 系统架构

### mach-O 文件
 macOS 和 iOS 和可执行文件。主要分为了三大块，Header、Load Command 和 Section。Header 的作用是配置和说明，Load Command 说明了需要加载的库、流程和数据，Section 是 Load Command 的详细描述。

 