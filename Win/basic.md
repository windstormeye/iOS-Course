## win

### 程序包

#### exe

#### WPF
Windows Presentation Foundation

#### UWP
Universal Windows Platform，该类型应用本质上也是一个 exe，但会经过一层包装，需要被微软统一审核，使用受限的 API。

有些 win32 程序重写比较费劲，但可以通过 win32 转译到 UWP，同样给其装上一个容器，使其运行在其中。

##### 缺点

* 添加入口进右键菜单需要 UAC 提权。

