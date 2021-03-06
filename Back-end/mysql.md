# Mysql
## Mysql 的客户端/服务器架构
`mysql` 的服务器进程默认名称为 `mysqld`，常用的 `mysql` 客户端进程的默认名称为 `mysql`。

## 启动
`mysql -h主机名 -u用户名 -p密码`

* `-h/--host=主机名`：后可跟服务器域名或 IP 地址，若 mysql 服务器进程运行在本机则可以省略；
* `-u/--user=用户名`；
* `-p/--password=密码`。

### 如果 mysql 默认端口号 3306 被占用
在启动 `mysql` 服务器时使用 `mysqld -P3307`，启动 `mysql` 客户端程序时使用 `mysql -u root -P3307 -p`，打写的 `-P` 代表选择

## 服务器处理客户端请求
### 连接管理
每当有一个客户端连接到 `mysql` 服务器进程时，服务器进程都会创建一个线程来处理与客户端的交互。当该客户端与服务器断开连接时，服务器并不会立即把该线程销毁，而是缓存起来供下一次使用。但线程开辟太多会影响系统性能，需要限制同时连接服务器的客户端数量。

### 查询缓存
`mysql` 服务器会把已经处理过的查询请求和结果缓存起来，若下一次有相同的请求时，则从缓存中直接返回。但如果两个查询请求在任何自负上的不同，都会导致缓存不会命中。

`mysql` 缓存系统会检测当前缓存中设计到的每张表，只要该表的结果或者数据被修改，则该表的所有高速缓存都将变为无效并删除。**从 mysql 8.0 中已移除**。


## 存储引擎
表，是有一行一行的记录组成的，但这只是逻辑上的概念，在物理上如何表示记录，怎么从表中读取数据，怎么把数据写入具体的物理存储器上，这都是存储引擎做的事情。我常用的 mysql 存储引擎为具备外键支持的事务存储引擎 **`InnoDB`** 。

其它常见的 `mysql` 存储引擎有：

存储引擎 | 描述
---- | ---- s
MEMORY | 置于内存的表
MyISAM | 主要的非事务处理存储引擎


### 设置表的存储引擎
存储引擎负责对表中的数据进行提取和写入工作的，可以**为不同的表设置白虎通你的存储引擎**，不同的表可以有不同的物理存储结构，不同的提取和写入方式。

## 启动选项和配置文件
* mysql 服务器默认的客户端连接数量为：151；
* 表的默认存储引擎为：`InnoDB`；
* 通过 `-h` 方式来启动服务器程序时，客户端和服务器进程之间使用 `TCP/IP` 网络进行通信，如果想要禁止这种方式，可以在启动服务器命令选项中加上 `--skip_networking` 来禁止使用 `TCP/IP` 方式来进行连接。




