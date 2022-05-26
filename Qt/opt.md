## processEvents
`QCoreApplication::processEvents()` 可以解决 `while(1)` 这种卡死当前线程的事情

有些情况下，我们并不想卡死 UI 线程的操作，但又必须等待某件事的完成，用户此时可以选择不等待直接关闭程序或停止该逻辑的执行。在 Qt 中可以使用 `processEvents` 进行。详见[文章](https://www.qter.org/forum.php?mod=viewthread&tid=1838)


## Loader
Loader 用于延迟加载组件，例如登录等，详见[文章](https://www.cnblogs.com/linuxAndMcu/p/11960251.html)