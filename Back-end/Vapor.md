# Vapor
在这里将记录使用 Vapor 的过程中遇到的问题。感觉特别一些设计模式的 tips 杂糅在一起后，就特别像 `Django`。

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

### 对表字段的修改
`Vapor` 没有像 `Django` 那么强大的工作流，很多人都说 `Perfect` 像 `Django`，我自己的认为 `Vapor` 像 `Flask`。

对 `Vapor` 修改表字段，不仅仅只是修改 `Model` 属性这么简单，同样也不像 `Django` 中修改完后，执行 `python manage.py makemigrations` 和 `python manage.py migrate` 就结束了，我们需要自己创建迁移文件，自己写清楚此次表结构到底发生了什么改变。

在泊学的[这篇文章](https://boxueio.com/series/vapor-fluent/ebook/473)中推荐在 `App` 目录下创建一个 `Migrations group`，方便操作。但我思考了一下，这么做势必会造成 `Model` 和对应的迁移文件割裂，然后在另外一个上级文件夹中又要对不同迁移文件所属的 `Model` 做切分，这很显然是有一些问题的。最后，我脑子冒出了一个非常可怕的想法：“`Django` 是一个非常强大、架构非常良好的框架！”。

所以，最后我的目录是这样的：

```shell
Models
└── User
    ├── Migrations
    │   ├── 19-04-30-AddUserCreatedTime.swift
    │   └── 19-04-30-DeleteUserNickname.swift
    ├── UserController.swift
    └── User.swift
```

这是 `Django` 中的一个 `app` 文件树：
```shell
user_avatar
├── __init__.py
├── admin.py
├── apps.py
├── migrations
│   ├── 0001_initial.py
│   ├── 0002_auto_20190303_2154.py
│   ├── 0002_auto_20190303_2209.py
│   ├── 0003_auto_20190303_2154.py
│   ├── 0003_auto_20190322_1638.py
│   ├── 0004_merge_20190408_2131.py
│   └── __init__.py
├── models.py
├── tests.py
├── urls.py
└── views.py
```

已经删除掉了一些非重要信息。可以看到，`Django` 的 `app` 文件夹结构非常好！注意看 `migrations` 文件夹下的迁移文件命名。如果开发能力不错的话，我们是可以做到与业务无关的 `app` 发布供他人直接导入到工程中。

不过关于工程文件的管理，这是一个智者见智的事情啦～对于我个人来说，我反而更加喜欢 `Vapor`/`Flask` 一系，因为需要什么再加什么，整个设计模式也可以按照自己的喜好来做。

#### 删除一个表字段
使用 `Swift` 开发服务端很容易受到使用 `Swift` 做其它开发的影响。刚开始时我确实认为在 `Model` 中把需要删除的字段删除就好了，然而运行工程后去查数据库发现并不是这么一回事。

首先，我们需要先创建一个文件来写 `Model` 的迁移代码，但这不是必须的，你可以把该 `Model` 后续需要进行表字段的 CURD 都写在同一个文件中，因为没一个迁移都是一个 `struct`。我的做法是像上文所说，对每一个迁移都做新文件，并且每一个迁移文件都写上“时间”和“做了什么”。

```swift
import FluentMySQL

struct DeleteUserNickname: MySQLMigration {
    static func prepare(on conn: MySQLConnection) -> EventLoopFuture<Void> {
        return MySQLDatabase.create(User.self, on: conn, closure: {
            $0.field(for: \.id, isIdentifier: true)
            $0.field(for: \.nickname)
        })
    }
    
    static func revert(on conn: MySQLConnection) -> EventLoopFuture<Void> {
        return MySQLDatabase.delete(User.self, on: conn)
    }
}
```

#### 增加/修改一个表字段
```swift
import FluentMySQL

struct AddUserCreatedTime: MySQLMigration {
    static func prepare(on conn: MySQLConnection) -> EventLoopFuture<Void> {
        return MySQLDatabase.update(User.self, on: conn, closure: {
            $0.field(for: \.fluentCreatedAt)
        })
    }
    
    static func revert(on conn: MySQLConnection) -> EventLoopFuture<Void> {
        return MySQLDatabase.delete(User.self, on: conn)
    }
}
```

需要注意的是，不管你是要做 CURD 中的任何一个功能，你都需要实现 `prepare` 和 `revert` 两个方法，`revert` 方法的作用是用于撤销 `prepare` 方法中的逻辑。
