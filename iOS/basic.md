# iOS 基础
强推 [Apple Developer Documentation](https://developer.apple.com/documentation/)

## 名词解释
* ipa：iPhone application archiv；

## 使用不同版本的 `Cocoapods`
很多时候不同项目都会使用不同的 `pod` 版本进行项目管理，比如我公司的项目选择了 `1.3.1` 版本的 `pod`，而我自己电脑上的是 `1.5.3` 版本，这样势必会造成使用 `pod` 各种命令时出现问题，因此可以使用 `Bundler` 这个 `ruby` 版本管理工具。

### 下载
```shell
gem install bundler
```

若提示：
```
Fetching: bundler-2.0.1.gem (100%)
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /usr/bin directory.
```

可以使用该命令：
```shell
sudo gem install -n /usr/local/bin bundler  
```

### 进入目录
进入到带有 `Podfile` 文件的目录。

### 创建 `Gemfile`
```shell
bundle init
```

### 设置版本
```ruby
source "https://rubygems.org"
# 或其它你的版本
gem 'cocoapods','1.3.1'
```

### 安装
```shell
bundle install
```

### 运行 `pod install`
```shell
bundle exec pod install
```

## iOS 进程间通信（IPC）
### `URL Scheme` 和 `openURL` 方法
需要注意的是：iOS 系统应用的 `URL scheme` 无法重复注册，但是对于其它应用来说，结果是「未定义」，官方说法：

> 如果有多个第三方应用程序注册同一个 `URL scheme`，那么无法确定将 `scheme` 传递给哪一个应用程序。

### 通用链接
在 iOS 9 引入了通用链接的原因之一，解决 `URL  Scheme` 劫持问题。在 Xcode 中打开 `Associated Domains` 选项。

### `UIActivity` 共享数据

### 应用程序扩展
应用程序扩展并不是应用程序，它们必须被捆绑到某个应用程序中，也就是「容器应用」，使用扩展的第三方应用程序，也就是「宿主应用」，可以与容器应用内的扩展程序包进行通信。但容器应用本身并不能直接与扩展应用通信。

### 剪贴板
需要注意的是：通用剪贴板在所有应用程序之间都是共享的，可以被设备砂锅的任何进程读取信息。

