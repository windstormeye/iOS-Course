# Vapor
在这里将记录使用 Vapor 的过程中遇到的问题。

## 如何快速开始
### 下载 `vapor`
[详见官网](https://docs.vapor.codes/3.0/install/macos/)。

### 运行 `Hello, world!`
* `vapor new yourProjectName`。创建模版工程，当然可以加上 `--template=api` 来创建提供对应服务的模版工程，但我测试了一下好像没什么区别。
* `vapor xcode`。创建 Xcode 工程，特别特别慢，而且会有一定几率失败。


### MVC
`Vapor` 默认是 `SQLite` 的**内存**数据库。我原本想看看 `Vapor` 自带的 `SQLite` 数据库中的表，但没翻着，最后想了一下，这是内存数据库啊，也就是说，每次 `Run` 数据都会被清空。可以从 `config.swift` 中看出：

```swift
// ...
let sqlite = try SQLiteDatabase(storage: .memory)
// ...
```

在 `Vapor` 文档中写了推荐使用 `Fluent` ORM 框架进行数据库表结构的管理，刚开始我们并不了解关于 `Fluent` 的任何内容，可以查看模版文件中的 `Todo.swift`：

```swift
import FluentSQLite
import Vapor


final class Todo: SQLiteModel {
    /// 唯一标识符
    var id: Int?
    var title: String

    init(id: Int? = nil, title: String) {
        self.id = id
        self.title = title
    }
}

/// 实现数据库操作。如增加表字段，更新表结构
extension Todo: Migration { }

/// 允许从 HTTP 消息中编解码出对应数据
extension Todo: Content { }

/// 允许使用动态的使用在路由中定义的参数
extension Todo: Parameter { }
```

从模版文件中的 `Model` 可以看出来创建一张表结构相当于是**描述一个类**，之前有使用过 `Django` 的经验，看到 `Vapor` 的这种 ORM 这么 `Swifty` 确实眼前一亮。`Vapor` 同样可以遵循 `MVC` 设计模式进行构建，在生成的模版文件中确实是基于 `MVC` 去做的。

####################  没写完


### 从 `SQLite` 到 `Mysql`
这部分 `Vapor` 官方文档讲的不够系统，虽然都点到了但是过于分散，而且我感觉 `Vapor` 的文档是不是跟 Apple 学了一套，细节都不展开，遇到一些字段问题得亲自写下代码，然后看实现和注释，不写之前你是很难知道在描述什么。



首先，在 `Package.swift` 中写下对应的依赖，

```swift
// swift-tools-version:4.0
import PackageDescription

let package = Package(
    name: "Unicorn-Server",
    products: [
        .library(name: "Unicorn-Server", targets: ["App"]),
    ],
    dependencies: [
        .package(url: "https://github.com/vapor/vapor.git", from: "3.0.0"),
        .package(url: "https://github.com/vapor/fluent-mysql.git", from: "3.0.0"),
    ],
    targets: [
        .target(name: "App",
                dependencies: [
                    "Vapor",
                    "FluentMySQL"
            ]),
        .target(name: "Run", dependencies: ["App"]),
        .testTarget(name: "AppTests", dependencies: ["App"])
    ]
)
```

