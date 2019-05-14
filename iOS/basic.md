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