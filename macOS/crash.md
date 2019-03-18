# 解析 crash log（一）
## 前言
在负责的产品中有最近一段时间有极个别用户老是反馈有偶尔闪退的情况，而且就这几个用户反复出现，其它用户，甚至就坐在他边上的用户进行了一样的操作都没有任何问题。

刚开始丢了个重现构建的新包给这几位用户将就的用着，但直到今天 PM 受不了了，让我无论如何都要想办法解决这个问题，但我也一脸懵逼啊，按照用户的操作路径来看我也没能复现，甚至分析平台都抓不到对应信息，更何况这还是个企业级应用，没上到 AppStore，也就没法从 iTunes Connect 中拿到崩溃日志。

所以开始酝酿了一个事情......

## 分析问题
为了快速定位到问题所在，PM 和 leader 再三的跟用户进行交流，大家都没有一个比较好的方案，最后我厚着脸皮让用户照着下面这张图从设备中导出了一份崩溃日志发送给我：

![图1 从设备中导出日志的步骤](https://i.loli.net/2019/03/14/5c89de148f448.jpg)

从真机中拿到这份 `ips` 文件后就好办了很多，如果此时直接打开这个文件，可以看到如下图所示内容：

![图2 未符号化内容](https://i.loli.net/2019/03/14/5c8a1c56e55d7.png)

能够拿到真机的情况下，
* 在不删掉 app 的情况下直接调试；
* 无法复现问题，则把真机插入电脑，打开 Xcode -> Window -> Devices and Simulators -> View Device Logs，直接找到需要的 log；

在拿不到真机的情况下，先来直接讲解一个能够解决问题的流程：

### 第一步：`ips` 文件
“死皮赖脸”的让用户通过图 1 所示方法导出一份最新的 `.ips` 文件，并让用户分享给自己，并把文件名修改为 `.crash` 后缀，使其标识为 `crash` 类型；

### 第二步：`.dSYM` 文件
`.dSYM` 文件（debugging SYMBols，调试符号表）。从打包机（如果是通过打包机隔离构建的话）或本机上导出一份与用户设备中安装的 app 版本一致的 `.dSYM` 文件，该文件中详细的记录了 16 进制下的函数地址的映射信息。

需要注意的是，Xcode 的默认设置是会在 release 和 debug 环境下已经配置好了 archive 时自动带出 `.dSYM` 文件，如果你发现打开包内容时并没有发现 `.dSYM` 文件，可以到 Xcode 的 `Build Settings` 中查看 `Debug Infomation Format` 字段的配置进行修改。

`.dSYM` 文件对于后续排查问题十分重要，每一次 release 版本都最好要保存对应的 `dSYM` 文件或把整个 `Archives` 文件进行保存；


### 第三步：`symbolicatecrash` 工具
`symbolicatecrash` 工具。该工具跟随 Xcode，是获取符号化结果的最方便工具。`symbolicatecrash` 的地址视 Xcode 的安装路径而定，大致的地址为：

```shell
你的Xcode安装路径/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
```

### 第四步：获取崩溃代码信息
有了 `.ips` 文件、`.dSYM` 文件和 `symbolicatecrash` 工具后就可以直接进行解析日志了～因为我把这三个文件都统一放在了一个文件夹中，所以实际上命令看起来会是这样：

`./symbolicatecrash ./yourApp.crash ./yourApp.dSYM > crash.log`

如果在输入这条命令时告知找不到 `DEVELOPER_DIR`，可以导出一份：

```
export DEVELOPER_DIR="你的Xcode安装位置/Xcode.app/Contents/Developer"
```

对输出的结果重定向到了当前路径下的 `crash.log` 中，此时打开该文件，看到的内容是这样的：

![未完全符号化](https://i.loli.net/2019/03/14/5c8a29e11f031.png)

很崩溃啊！重要的细节一个都没有展示出来！随后我换了一个思路，我们重新回到第三步

### 回到第三步：`atos` 命令
关于 `atos` 命令的描述，可以使用 `man atos` 查看具体信息：

```
...
DESCRIPTION
     The atos command converts numeric addresses to their symbolic equiva-
     lents.  If full debug symbol information is available, for example in a
     .app.dSYM sitting beside a .app, then the output of atos will include
     file name and source line number information.
...
```

这是一个专用于 macOS 的控制台工具，从描述中可以看出，可以将地址转换为实际二进制图像的符号化字符串（实际代码）。上文中说到的 `symbolicatecrash` 工具是 apple 基于 `atos` 方便开发者进行的优化封装，但不知为何我的 `symbolicatecrash` 并不能完整的符号化所有 crash log 中的内容。所以现在将直接使用 `atos` 进行符号化，操作稍微繁琐一些，我们可以把对应的命令修改为：

`atos -o yourApp.dSYM/Contents/Resources/DWARF/yourApp -arch arm64 -l 0x104e40000 0x0000000104f90198`

`0x104e40000` 和 `0x0000000104f90198` 这两个地址是什么意思呢？我们再看这张图：

![找到需要查看的地址](https://i.loli.net/2019/03/15/5c8b04efd072c.jpg)

`0x104e40000` 为 `local address`，`0x0000000104f90198` 为 `address`，这两个地址在不借助其它工具的前提下可以直接手算出来，如果你对手算地址感兴趣的话，可以看[这篇文章](http://foggry.com/blog/2015/07/27/ru-he-shou-dong-jie-xi-crashlog/)。

通过执行上述 `atos` 命令，可以看到输出了正确的信息，如下图所示：

![正确的崩溃信息](https://i.loli.net/2019/03/15/5c8b06cb81359.jpg)

接下来就可以把多个地址进行解析，配合着在堆栈中的这几个关键信息基本上就可以定位到具体的 crash 代码文件和行数。


### 其它工具
关于符号化奔溃报告，还有以下几种：
* `dwarfdump`。这个工具用来应付普通的 crash 日志符号化完全是小题大做，但不排除某些极端下的情况。这个我没使用过，不做展开。
* `lldb`。`lldb` 在日常使用 Xcode 进行开发的过程中已经非常熟悉了，是 Xcode 的默认调试器。这部分我也没试过，大家可以自行搜索进行尝试。



### 总结
总不能每次出问题都要经过上述这么费劲的流程吧？当然可以说把这些流程写成一个脚本，每次只需要输入 `local address` 和 `address` 即可，但实际上这样的效果并不令人感到舒服。所以，为了节省后续更多的时间，接下来将准备写一个 Mac App，每次进行符号化时，只需要根据提示提前配置好对应的文件，再写下对应的 `local address` 和 `address` 就可以完成。

本系列下篇文章中将继续讲解......
