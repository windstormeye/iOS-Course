# iOS 和 macOS 性能优化

## 命令行工具
### `top`
`top` 命令能够连续动态地更新显示大量有关系统性能的参数。`top -u` 能够把 CPU 当前的占用比排序，置顶最活跃的进程。

### `time`
使用 `time` 命令测试程序的耗时。

### `simple`
查看程序的调用树。可以集合 `grep` 来过滤掉一些干扰数据。

### `getrusage()`
调用该方法与使用 `top` 和 `time` 命令获取的信息相同，但没有启动单独进程的开销。

## Instruments
### macOS 冷启动性能分析
当需要测试应用程序「冷启动」性能时使用它进行测试非常有效。冷启动是指启动和登录后的过程，此时 UI 程序还未驻留内存。但是启动 Instruments GUI 程序会加载大多数 GUI 框架，这意味着使用应用程序时，收集的任何信息不再代表冷启动。这种情况一般出现在 macOS 上。

Instruments 可以允许在不与 UI 交互的情况下启动性能分析会话。例如：

```shell
instruments -t "Time Profile" -l 2000 -D sumintsm-cpu.trace.sumintsm
```

通过启动以上命令启动 Instruments 命令行工具，使用 `Time Profile` 工具配置 2s，然后将结果数据写入跟踪文档 `sumintsm-cpu.trace`，如果文件已经存在，则追加数据。

命令行程序的占用空间比 GUI 应用程序小得多，不加载任何 UI 框架或资源。

