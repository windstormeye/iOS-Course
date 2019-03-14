# PJCrash 
## 前言
在负责的产品中有最近一段时间有极个别用户老是反馈有偶尔闪退的情况，而且就这几个用户反复出现，其它用户，甚至就坐在他边上的用户进行了一样的操作都没有任何问题。

刚开始丢了个重现构建的新包给这几位用户讲究的使用着，但直到今天 PM 受不了了，让我无论如何都要想办法解决这个问题，但我也一脸懵逼啊，按照用户的操作路径来看我也没能复现，甚至分析平台都抓不到对应信息，更何况这还是个企业级应用，没上到 AppStore，也就没法从 iTunes Connect 中拿到崩溃日志。

所以开始酝酿了一个事情......

## 分析问题
为了快速定位到问题所在，PM 和 leader 再三的跟用户进行交流，大家都没有一个比较好的方案，最后我厚着脸皮让用户照着下面这张图从设备中导出了一份崩溃日志发送给我：

![图1 从设备中导出日志的步骤](https://i.loli.net/2019/03/14/5c89de148f448.jpg)

从真机中拿到这份 `ips` 文件后就好办了很多，如果此时直接打开这个文件，可以看到如下图所示内容：

![图2 未符号化内容](https://i.loli.net/2019/03/14/5c8a1c56e55d7.png)

`ips` 文件的符号化目前我所致的方式有以下几种：
能够拿到真机的情况下，
* ~在不删掉 app 的情况下直接调试（废话）~；
* 无法复现问题，则把真机插入电脑，打开 Xcode -> Window -> Devices and Simulators -> View Device Logs，直接找到需要的 log；

在拿不到真机的情况下（我是这种情况），直接讲解一个能够解决问题的流程：

### 第一步：`ips` 文件
“死皮赖脸”的让用户通过图 1 所示方法导出一份最新的 `.ips` 文件，并让用户分享给自己，并把文件名修改为 `.crash` 后缀，使其标识为 `crash` 类型；

### 第二步：`.dSYM` 文件
`.dSYM` 文件（debugging SYMBols，调试符号表）。从打包机（如果是通过打包机隔离构建的话）或本机上导出一份与用户设备中安装的 app 版本一致的 `.dSYM` 文件，该文件中详细的记录了 16 进制下的函数地址的映射信息。`.dSYM` 文件对于后续排查问题十分重要，每一次 release 版本都最好要保存对应的 `dSYM` 文件或把整个 `Archives` 文件进行保存；

### 第三步：`symbolicatecrash` 工具
`symbolicatecrash` 工具。该工具的地址视 Xcode 的安装路径而定，大致的地址为：`你的 Xcode 安装路径/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash`，该工具是 Xcode 自带的一个隐藏分析工具，通过 `.ips` 文件和 `.dSYM` 文件来定位 app 崩溃的具体位置；

### 第四步：获取崩溃代码信息
有了 `.ips` 文件、`.dSYM` 文件和 `symbolicatecrash` 工具后就可以直接进行解析日志了～因为我把这三个文件都统一放在了一个文件夹中，所以实际上命令看起来会是这样：

`./symbolicatecrash ./yourApp.crash ./yourApp.dSYM > crash.log`

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

这是一个专用于 macOS 的控制台工具，从描述中可以看出，可以将地址转换为实际二进制图像的符号化字符串（实际代码），我们可以把对应的命令修改为：

`atos -o yourApp.dSYM/Contents/Resources/DWARF/yourApp -arch arm64 -l 0x104e40000 0x0000000104f90198`

`0x104e40000` 和 `0x0000000104f90198` 这两个地址是什么意思呢？我们再看这张图：



