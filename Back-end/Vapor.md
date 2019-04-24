# Vapor
在这里将记录使用 Vapor 的过程中遇到的问题。

## 如何快速开始
### 下载 `vapor`
[详见官网](https://docs.vapor.codes/3.0/install/macos/)。

### 运行 `Hello, world!`
* `vapor new yourProjectName`。创建模版工程，当然可以加上 `--template=api` 来创建提供对应服务的模版工程，但我测试了一下好像没什么区别。
* `vapor xcode`。创建 Xcode 工程，特别特别慢，而且会有一定几率失败。