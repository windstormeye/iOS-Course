# 来一次完整的使用 Playground（一）
## 简介
在使用 `Xcode Playground` 进行 WWDC 2019 scholarship 项目的开发时，原本以为 apple 指定的 Xcode 10.1 版本上会有不错的优化，但没想到跟两年前使用的效果出奇一致的差劲！！！让我不仅怀疑，“这是 Apple 开发的工具吗？”。从我知道 `Xcode Playground` 开始到现在一直都给我“这是一个神器”的感觉，曾经也尝试着在给社团同学讲解相关开发知识点时使用过 `Xcode Playground`，但每次想坚持尝试时都不得不放弃。

接下来，我将介绍一遍我在使用 `Xcode Playground` 中遇到的问题和解决思路。

## 怎么写啊？
我们创建一个 `Single View` 的工程，如下图所示：

![新建工程](https://i.loli.net/2019/03/18/5c8f5b13e8a57.jpg)

然后你会看到一个简洁明了的工程样式：

```Swift
import UIKit
import PlaygroundSupport

class MyViewController : UIViewController {
    override func loadView() {
        let view = UIView()
        view.backgroundColor = .white

        let label = UILabel()
        label.frame = CGRect(x: 150, y: 200, width: 200, height: 20)
        label.text = "Hello World!"
        label.textColor = .black
        
        view.addSubview(label)
        self.view = view
    }
}
// Present the view controller in the Live View window
PlaygroundPage.current.liveView = MyViewController()
```

OK，代码非常容易理解，如果你使用也是 Xcode 10.1 那么在 `PlaygroundPage.current.liveView = MyViewController()` 的左边，会看到一个“播放”键，点击它即可运行。

但运行后的界面呢？可以按照下图箭头所指向之处，选择视图展示方向：

![](https://i.loli.net/2019/03/18/5c8f5d43075a9.jpg)

一切正常！这跟之前毫无两样，就多了设置一下 `PlaygroundPage` 的活动视图而已。此时我们信心满满一定能够玩好 `Xcode Playground` ！结合按照以往的工程习惯，开始对项目进行分层，分模块进行项目的划分。

## 问题来了！
正准备按照文件夹的方式在 `Sources` 文件夹下划分模块，发现居然无法创建文件夹？？？是的，在 `Xode Playground` 无法直接创建文件夹，如果非要创建文件夹，可以在 `Finder` 中创建。

那好吧，既然没法方便的场景文件夹那就算了吧～估计 Apple 并不想我们在 `Xcode Playground` 中进行大工程的开发。这回终于可以开始写代码了！

那就先来把 `HomeViewController` 给写了吧。根据示例代码你写出来的代码可能会是这样：

```Swift
class PJHomeViewController: UIViewController {
    override func loadView() {
        view.backgroundColor = .red
    }
}
```

下一步在 playground 中使用它！

“嗯？怎么没有代码提示？好奇怪，算了吧～反正 Xcode 写 Swift 也会偶尔没有提示，习惯就好”——这是你在 playground 中写代码时内心可能和自己的对话。凭借着简单的类名，我们可以很快的写出以下代码：

```Swift
  
import UIKit
import PlaygroundSupport

PlaygroundPage.current.liveView = PJHomeViewController()
```

等了一会儿后......（可能是一分钟，可能是 10 秒，也可能 3 秒）

你收到了这样的一个提示：`Use of unresolved identifier 'PJHomeViewController'`。告诉你 `PJHomeViewController` 这个标识符不存在！

“不可能啊！这不可能啊！”，然后陷入了深深的怀疑当中，“难道我写了这么久的 iOS，居然连一个 `class` 都会创建错？”。反复查看代码后，你确实自己写的确实没有问题，开始从网上搜索答案。

终于，你找到了解决方案！`Sources` 中的类与 playground 并不位于同一 `Module` 下，我们需要把 `PJHomeViewController` 暴露出去。你把代码修改为了：

```Swift
import UIKit

public class PJHomeViewController: UIViewController {
    public override func loadView() {
        view.backgroundColor = .red
    }
}
```

然后再回到 playground 中，运行！嗯？怎么一直在转圈？重启 Xcode！嗯？怎么还是不行？检查代码，然后你发现了在 `loadView` 并没有给 `view` 设置 `frame`，看来是自己的学术不精。修改后代码为：

```Swift
import UIKit

public class PJHomeViewController: UIViewController {
    public override func loadView() {
        view = UIView(frame: CGRect(x: 0, y: 0, width: 375, height: 667))
        view.backgroundColor = .red
    }
}
```

此时再运行工程，我们终于看到了红色的画面！吐了一口气～

## 准备大干一场！
摸熟套路后，我们终于可以大干一场了！

在大干一场一场的过程中，你会被“代码提示怎么又没有了？！”“报错异常怎么还不消失？”“运行怎么需要这么久？”“文件删除后怎么一直没更新目录？”等问题困扰，所以！

我十分推荐大家优先使用 Xcode 进行源码的编写和调试，在 playground 中进行编码和调试都是一个“十分痛苦”的体验，如果再加上你的设备比较差劲的话，那 `Xcode Playground` 的“实时预览”的效果几乎可以说废掉了。

关于 `Xcode Playground` 的实现猜测是跑在了一个单独的进程中做文件差分编译，所以才会导致在某些情况下出现“卡死”甚至 Xcode 整个崩掉的情况。

但是这个情况在 iPad 上完全没有，可以尝试使用使用 `Swift Playground` 而不是 `Xcode Playground`。

## 总结
`Xcode Playground` 的优势很大，但就目前来看劣势在慢慢缩短，我在使用 2015 款的 15-inch 高配的 MBP 和 2017 款的 15-inch 高配两款机器下的 `Xcode Playground` 对老款机器十分不友好，让我举步维艰，最后放弃转而使用 Xcode 开发完再丢入  `Xcode Playground` 进行效果查看。

下篇文章中将完整的介绍我的 WWDC 2019 scholarship 项目完整内容。