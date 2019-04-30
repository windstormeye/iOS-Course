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

#### `Package.swift`
在 `Package.swift` 中写下对应的依赖，

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
        // here
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

触发更新

```shell
vapor xcode
```

`Vapor` 搞了我几次，更新依赖的时候特别慢，而且还更新失败，导致我现在每次更新时都要去确认一遍依赖是否更新成功。

#### 更新 ORM
更新成功后，我们就可以根据之前生成的模版文件 `Todo.swift` 的样式改成 `MySQL` 版本的 ORM：

```swift
import FluentMySQL
import Vapor

/// A simple user.
final class User: MySQLModel {
    /// The unique identifier for this user.
    var id: Int?
    
    /// The user's full name.
    var name: String
    
    /// The user's current age in years.
    var age: Int
    
    /// Creates a new user.
    init(id: Int? = nil, name: String, age: Int) {
        self.id = id
        self.name = name
        self.age = age
    }
}

/// Allows `User` to be used as a dynamic migration.
extension User: Migration { }

/// Allows `User` to be encoded to and decoded from HTTP messages.
extension User: Content { }

/// Allows `User` to be used as a dynamic parameter in route definitions.
extension User: Parameter { }
```

其实改动的地方只有两个，`import FluentMySQL` 和继承自 `MySQLModel`。

#### 修改 `config.swift`
```swift
import FluentMySQL
import Vapor

/// 应用初始化完会被调用
public func configure(_ config: inout Config, _ env: inout Environment, _ services: inout Services) throws {
    // === mysql ===
    // 首先注册数据库
    try services.register(FluentMySQLProvider())

    // 注册路由到路由器中进行管理
    let router = EngineRouter.default()
    try routes(router)
    services.register(router, as: Router.self)

    // 注册中间件
    // 创建一个中间件配置文件
    var middlewares = MiddlewareConfig()
    // 错误中间件。捕获错误并转化到 HTTP 返回体中
    middlewares.use(ErrorMiddleware.self)
    services.register(middlewares)
    
    // === mysql ===
    // 配置 MySQL 数据库
    let mysql = MySQLDatabase(config: MySQLDatabaseConfig(hostname: "", port: 3306, username: "", password: "", database: "", capabilities: .default, characterSet: .utf8mb4_unicode_ci, transport: .unverifiedTLS))

    // 注册 SQLite 数据库配置文件到数据库配置中心
    var databases = DatabasesConfig()
    // === mysql ===
    databases.add(database: mysql, as: .mysql)
    services.register(databases)

    // 配置迁移文件。相当于注册表
    var migrations = MigrationConfig()
    // === mysql ===
    migrations.add(model: User.self, database: .mysql)
    services.register(migrations)
}
```

注意 `MySQLDatabaseConfig` 的配置信息。如果我们的 mysql 版本在 **8** 以上，目前只能选择 `unverifiedTLS` 进行验证连接MySQL容器时使用的安全连接选项，也即 `transport` 字段。在代码中用 `// === mysql ===` 进行标记的代码块是跟模版文件中使用 `SQLite` 所不同的地方。

#### 运行
运行工程，打开 mysql 进行查看。

```shell
mysql> show tables;
+----------------------+
| Tables_in_unicorn_db |
+----------------------+
| fluent               |
| Sticker              |
| User                 |
+----------------------+
3 rows in set (0.01 sec)

mysql> desc User;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | bigint(20)   | NO   | PRI | NULL    | auto_increment |
| name  | varchar(255) | NO   |     | NULL    |                |
| age   | bigint(20)   | NO   |     | NULL    |                |
+-------+--------------+------+-----+---------+----------------+
3 rows in set (0.01 sec)
```

`Vapor` 不像 `Django` 那般在生成的表加上前缀，而是你 ORM 类名是什么，最终生成的表名就是什么，这点很喜欢！