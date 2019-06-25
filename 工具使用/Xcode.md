## Xcode
在这篇文章中主要讲述了我在使用 `Xcode` 过程中遇到的问题

### Xcode 简单入门
[Xcode 简单入门](./4_Xcode.md)

### 观察 App 启动时间
App 的启动时间分为“ `main` 函数加载之前的时间（`t1`）” 和“`main` 函数加载之后的时间（`t2`）” 组成的，对于 `t1` 时间来说，需要分析 app 的启动日志，打开这个日志方法如下所示：

Product -> Scheme -> Edit Scheme -> Run -> Arguments -> Enviouronment Variables -> 填入 `DYLD_PRINT_STATISTICS` ，设值为 `1` 。运行工程后，即可看到 `Xcode` 控制台中打印出：
```
platform initialization successful
Total pre-main time: 1.0 seconds (100.0%)
         dylib loading time: 739.36 milliseconds (73.7%)
        rebase/binding time:  24.84 milliseconds (2.4%)
            ObjC setup time:  92.96 milliseconds (9.2%)
           initializer time: 145.57 milliseconds (14.5%)
           slowest intializers :
             libSystem.B.dylib :   5.09 milliseconds (0.5%)
          libglInterpose.dylib :  71.06 milliseconds (7.0%)
```

* 从打印出的信息可以看出，我的这个项目的 `t1` 时间中的主要耗时是在动态库的加载上，而动态库又分为系统界别和开发者自己添加的动态库。一般系统级别的动态库加载时间我们几乎不用进行考虑，重点要放在我们自己添加的第三方动态库上。Apple 官方推荐 app 的启动时间最好控制在 400ms 内，我这个单是动态库加载就已经快 740ms 了 😅 ，当 app 20s 后还没有启动成功，则会被系统强杀掉也就是`8badf00d`，而解决动态库加载耗时多长的办法要么合并要么删除，我目前没有太多的经验；

* 减少 OC 类数量也是个办法，同样也是要么合并要么删除。这样做可以加快动态链接，降低重定位/绑定所耗费的时间；
* 还可以使用 `initializer` 替换 `load` 方法，或是尽量将 `load` 方法中的代码尽量延后（懒加载），对象的初始化所耗费的时间会减少。但是我从未使用过 `load` 方法哈哈。

而对于 `t2` 时间来说，其主要关注的是第一个页面构建并完成的时间，比如在 `viewDidLoad` 和 `viewWillAppear` 这两方法中做的事情要尽可能简单。

### 断点行为
可以创建一些断点，只在某些条件下才触发，并且在命中断点时执行一些复杂操作。



