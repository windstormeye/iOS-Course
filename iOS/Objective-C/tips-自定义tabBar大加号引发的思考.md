## 前言
最近在琢磨一些之前非常想玩的点，最让我感到有趣的是天朝才有的 tabBar 中间大加号，而且我还发现好像咸鱼的没做好，如果我们去点击咸鱼中间的加号上边部分，会发现其实是没有响应的，必须要点进 tabBar 里边才能被响应到。其实这个需求我自己之前也有遇到过，有些时候我会撸码撸眼花了，添加一个 button 在 bottomView 上后，y 坐标跑到了 superView 的顶部以上的区域，然后死活响应不了任何的点击事件，郁闷了许久后才发现原来是自己的坐标设置错了😑。

事后我就在想，为什么会出现响应不了的情况呢？明明已经 addSubview 到对应的视图中去了呀，在 iOS 中视图能够响应用户的触摸事件主要有以下几种情况：
1. 视图是不隐藏的：`self.hidden = NO`
2. 视图是允许交互的：`self.userinterfactionEnable = YES`
3. 视图透明度大于 `0.1：self.alpha > 0.01`
4. 视图包含这个点 `pointInside:withEvent: = YES`

而我之前犯的错误就在于第四点，添加上去的 button 所在父视图不包含这个点。如果想要解决我之前遇到的问题，那就要重写父视图的 hitTest:withEvent: 方法，并且还要在其中做坐标转化。 

## 那，hitTest:withEvent: 方法是什么？
想要了解 `hitTest` 方法是做什么的，就得先了解该方法作用在哪个对象上， `Xcode` 的帮助文档是这么描述的：`Returns the farthest descendant of the receiver in the view hierarchy (including itself) that contains a specified point.`，是说通过这个方法可以返回触摸点所触摸到的视图以及包含该视图的所有视图（直译过来真是太拗口了，我意译了），再简单一些来说，用我之前的例子，如果我现在点击了这个 `button` ，那么我在这个方法中就可以获取到包含这个 `button` 视图的所有父视图（包括爷爷视图、祖父视图）。在这个方法中返回的 `View` 是本次点击事件中需要的响应的 `View` ，也就是说，我们理论上可以在这个方法中返回任何想要响应该点击事件的视图。

其实到到这块问题已经解决得差不多了，我们只需要在 `hitTest:withEvent:` 方法中拦截到 `tabBar` 的用户点击事件，判断一下如果当前的 `point` 是落在了我们需要响应的视图上，就返回这个视图，否则都直接 `return [super ...]` ，让父视图自己去找最佳响应视图，如果父视图也找不到，那就丢给 `controller` 的 `view` ，如果 `controller` 的 `view` 也没找到，那就丢给 `controller` 所在的 `window` ，如果 `window` 也不处理，那就给 `UIApplication` ，到了 `UIApplication` 这一层真的还没人处理的话，那就直接丢了，不过我觉得应该没有这么极端的情况吧。

其实调用 `hitTest:withEvent` 方法时实际上是去调了 `pointInside:withEvent:` 这个方法注意（这两个方法都可以实现需求，为了方便调试，我就不删掉了），请看调用栈。

<img src="https://i.loli.net/2018/08/10/5b6d8e72bf006.jpg" width = "70%" height = "70%" align=center />

触发 `pointInside:withEvent:` 方法是在调用 `[super hitTest:withEvent]` 方法时，因为之前也说过了，当我们调用 `super` 方法实际上是让系统自己去找合适的响应视图，那系统是怎么找合适的响应视图？就是通过 `pointInside:withEvent:` 方法去寻找，如果我们不对 `pointInside:withEvent:` 方法的选择做干扰的话是按照系统的查找方法去做的。所以我们也可以直接在 `pointInside:withEvent:` 方法中做判断。

仔细一看，其实会发现 `hitTest:withEvent` 方法被调用了两次，是因为 `touch` 方法的 `begin` 和 `end` 所调用的，其实我们也就能看出一点，事件的传递其实是来源于触摸状态的变化（相当于没说😑，手机就是事件驱动的完全体）

OK，那问题就回到了如何判断 `point` 是否落在了我们需要响应的视图上，如果我们还是按照之前的想法去做，直接判断 `point` 的 `x` 和 `y` 是永远也得不到我们想要的结果，因为是按照系统坐标去计算的 `point` （不知道这么说大家能不能理解），我们需要做的就是把这个 `point` 坐标转化到某一个视图上，通过对这个视图做相对坐标来判断，而不是系统的绝对坐标。那 `iOS` 中如何进行坐标转换呢？系统中提供了这四个方法，这四个方法其实只需要弄懂其中的两个方法即可，其它的两个方法都是与之对应的。

```swift
public protocol UICoordinateSpace : NSObjectProtocol {
    
    @available(iOS 8.0, *)
    public func convert(_ point: CGPoint, to coordinateSpace: UICoordinateSpace) -> CGPoint
    @available(iOS 8.0, *)
    public func convert(_ point: CGPoint, from coordinateSpace: UICoordinateSpace) -> CGPoint
    @available(iOS 8.0, *)
    public func convert(_ rect: CGRect, to coordinateSpace: UICoordinateSpace) -> CGRect
    @available(iOS 8.0, *)
    public func convert(_ rect: CGRect, from coordinateSpace: UICoordinateSpace) -> CGRect
    
    @available(iOS 8.0, *)
    public var bounds: CGRect { get }
}
```

在这四个方法中，我觉得有点比较拗口的地方，但是只要理解好了 to 和 from 这两个介词就没关系了。

`public func convert(_ point: CGPoint, to coordinateSpace: UICoordinateSpace) -> CGPoint`

比如这个方法，注意是介词 to ，我们假设是这么调用

`self.convert(point, to: centerButton)`

文档是这么说的：“Converts a point from scene coordinates to view coordinates.”（这其实是 SpriteKit 的解释，不过都差不多），直译是“将一个点从场景坐标转换为视图坐标”，人话就是说：“把 point 这个点坐标从 self 视图坐标上转换成在 centerButton 视图坐标上”，也就是坐标原点从 self 视图的左上角变为了 centerButton 的左上角。经过这么一变换 point 坐标的值就约束在了 centerButton 的坐标系中，只要 point 坐标是在 centerButton 上就一定会被接收，当然这部分也需要做个 point(inside:with) 的判断。

另外一个方法，

`public func convert(_ point: CGPoint, from coordinateSpace: UICoordinateSpace) -> CGPoint`

这个方法就是上边 to 的反例，我是这么用的，

`centerButton.convert(point, from: self)`

翻译过来就是：“把来自 self 视图坐标系的 point 坐标转换成 centerButton 坐标系的坐标”。由上所诉，我们可以思考一下 `hitTest:withEvent` 的内部实现大概逻辑是怎么样的：
1. 先判断当前控件能否接受事件，满足这四个条件：可 `userInterfaceEnabled`、没有 `hidden` 、`alpha > 0.01` 、且点在当前控件内

2.从当前视图的根开始判断，举个例子：

<img src="https://i.loli.net/2018/08/10/5b6d94e7a8fb7.png" width = "70%" height = "70%" align=center />

用户当前的触摸点在紫色位置，首先 `A` 视图接收到了触摸事件（其实应该先是 `keyWindow` ），`A` 开始轮询自己的子视图，问到 `B` 中，问 `B` 在不在这个点在不在你里面，`B` 说在，`B` 开始轮询自己的子视图，问到了 `C` ，这个点在不在你里面，`C` 说在，`C` 开始轮询自己的子视图，`C` 发现除了自己没有子孙视图了，OK，那这个触摸事件就给 `C` 了，把 `C` 视图给 `return` 回去。所以大家可以发现，这其中实际上是从根视图一步步问下来的。为了更加详细的描述 `hitTest:withEvent` 方法的流程又画了张图做讲解（因为真的很有趣）：

假设现在视图变成了这样，白圈代表用户触摸的位置，

<img src="https://i.loli.net/2018/08/10/5b6d952f1dccd.png" width = "70%" height = "70%" align=center />

`hitTest:withEvent` 所采用的是 逆前序深度遍历，也就是先访问根节点，然后从该树索引最大的子视图到最小的子视图进行遍历，也即从右到左进行遍历。默认最左子树的最左叶节点为 `0` 。

<img src="https://i.loli.net/2018/08/10/5b6d95661bf8c.png" width = "70%" height = "70%" align=center />

当遇到上图所示的，用户的触摸位置为两个视图的重叠之处时，根据该算法将得到最右子树中的最深视图，而该视图就是距离屏幕最近的视图（好像这么说不太对），如下图所示，经过计算，我们最终将得到 `B-a` 这个视图。原本还想写一写伪代码，但是发现好像上面的那段大白话解释得更清楚一些，而且本来这部分代码苹果并没有开源，大家都只是猜测。

<img src="https://i.loli.net/2018/08/10/5b6d95a079c32.png" width = "70%" height = "70%" align=center />

总结一下，如果我们也想玩 `tabBar` 中间凸起的大按钮，按照正常逻辑去做，凸出的部分是肯定接收不到用户的触摸事件的。如果我们想要让凸出部分也接收到的用户的触摸部分，需要把用户在凸出的点击事件在 `pointInside:withEvent:` 或 `hitTest:withEvent` 进行拦截，在拦截的代码中做坐标的转换，判断是否用户的触摸点真的位于凸出位置即可。但是要注意，因为我们是直接自定义了一个 `tabBar` ， 而且还是直接添加上了一个 `button` ，在 `push` 到下一个页面的时候，对应的凸出的区域还是能点击的，因为 `tabBar` 虽然按照我们最初的想法是直接 `hidden` ，但是生命周期还在，所以触摸区域也生效。

## 再深挖一下
再往下挖一下，`iPhone` 是怎么知道用户的点击事件的呢？用户在屏幕上这个物理的点击事件是怎么传递到我们对应的 `App` 中去的呢？

`CPU` 处于睡眠状态，等待事件驱动（直接理解为待机 好了 😑）

↓↓↓↓↓

用户手指触摸屏幕

↓↓↓↓↓

屏幕收到触摸事件的响应，将该响应事件传递给 `IOKit `

↓↓↓↓↓↓

`IOKit` 把该触摸事件封装成了 `IOHIDEvent` 对象（这其中包括了所有的基本信息比如几根手指、时间戳等等信息，但基本上都是 `data` ，详见苹果代码 https://opensource.apple.com/source/IOHIDFamily/IOHIDFamily-308/IOHIDFamily/IOHIDEvent.h.auto.html）。该对象由 `IOHIDService` 生成，与之对等的还有 `IOHIDDiplays`

↓↓↓↓↓

`IOKit` 通过 `IPC` 将对象转发给 `SpringBoard.app` ，这个就是系统桌面进程了，它接收物理按键、触摸、加速、传感器等等 `Event` ，记得之前有人在越狱的 `iPhone` 上做了各种高大上的桌面，就是通过替换了这个进程，不过建议大家不要自己玩，`SpringBoard.app` 进程是唯一一个不能通过 `Home` 键和电源键进行开关机重启的。实际上严格来说应该是 `IOKit` 把这个对象发给了 `UIKit` ，`UIKit` 判断出了给它的是 `IOHIDEvent` 类型的对象，判断出了应该交由 `SpringBoard.app` 进行处理，最后才通过 `mach port`（进程端口）转发给 `SpringBoard.app` 。其实也就是说，不管用户在屏幕上做了什么操作，比如刷微博啦、发短信啦等等这些操作，也只有到了 `SpringBoard.app` 这一层才能进行才处理，之前进行的操作都是判断、封装和转发。说明一下：`mach port` 是 `iOS` 中多进程的一种方式，其它还有 `Distributed Notification` 、`Distributed Objects` ， `XPC` 等等方法

↓↓↓↓↓

`SpringBoard.app` 也就是桌面的主线程 `RunLoop` 收到 `IOKit` 转发来的消息后 `awake` ，并触发对应的 `mach port` 进行 `Source1` 回调， `__IOHIDEventSystemClientQueueCallback()`。`RunLoop` 的 `Source1` 回调，该回调由 `RunLoop` 和内核管理，`mach port` 驱动，如 `CFMachPort`（非常类似 `pipe` ，Mac下的 `NSPort` ）、`CFMessagePort`。

↓↓↓↓↓

如果 `SpringBoard.app` 监测到如果有 `app` 在前台运行（假设为 `pjhubs.app` ），`SpringBoard.app` 通过 `mach port` 转发给 `pjhubs.app` 。如果监测到前台有 `app` 在运行，则 `SpringBoard.app` 将触发 `Source0` 回调进入 `App` 内部响应阶段。`Source0` 回调处理 `App` 内部事件、`App` 自己负责管理（触发），如 `UIEvent`、`CFSocket` 等。

↓↓↓↓↓

`App` 主线程 `RunLoop` 接收到 `SpringBoard.app` 转发来的消息 `awake` ，注意这里 `App` `RunLoop` 不是 `SpringBoard.app` 的 `RunLoop` ，每一个进程都有自己主线程 `RunLoop` ，互不影响（其它平台都有的，只不过叫法不一样）。`awake` 后触发对应的 `mach port` 的 `source1` 回调 `__IOHIDEventSystemClientQueueCallback()`

↓↓↓↓↓

`Source1` 回调内部触发了 `Source0` 回调  `__UIApplicationHandleEventQueue()` 。

↓↓↓↓↓

`Source0` 内部回调，把 `App` 之前接收到的 `IOHIDEvent` 对象重新封装成了 `UIEvent` 对象（能够拿到事件类型和产生时间），一个事件将产生一个 `UIEvent` 对象，举个例子，如果用户的两个手指同时触摸在 `View` 上，那么 `View` 只调用一次 `touchBegin` 方法，但是 `allTouches` 参数将返回两个 `UITouch` 对象；如果用户是一前一后分别触摸 `View` 上，那么将分别调用两次 `touchBegin` ，但是用户两次都触摸到了同一个点上，也就是双击，这将只会创建一个 `UITouch` 对象，并且在第二次触摸时更新之前的 `UITouch` 对象 `tapCount` 属性进行加一操作。 `allTouches` 属性将返回一个 `UITouch` 对象，这个对象就是用于描述 `app` 和单个用户交互的对象。`UIEvent` 的事件类型，苹果归类成了三种：触摸事件、加速计事件和远程遥控事件。

↓↓↓↓↓↓

`Source0` 回调 内部去调用 `UIApplication` 对象的 `sendEvent：`方法，将 `UIEvent` 对象传给 `UIWindow` ，丢给 `UIWindow` 后，接下来的事情就是上文中我们已经做过的了。一般来说触摸事件添加到 `UIApplication` 的事件管理队列中（注意不是栈，因为先产生的事件要先处理，要不然就乱了）后，`UIApplication` 会在事件的最前端取出事件，然后分发下去，通常会先给 `keyWindow` 进行处理，通过 `keyWindow` 进行一层层的往下分发，流程可以概括为：`UIApplication` → `UIWindow` → `SuperView` → `SubView`

↓↓↓↓↓

通过递归调用 `UIView` 层级的 `hitTest：with：`和 `point ( inside:with: )`方法找到 `UIEvent` 中每个 `UITouch` 所属的 `UIView` ，因为一个 `UIEvent` 事件上是拥有多个 `touchs` 的（毕竟多点触控嘛），找到这个所属的 `UIView` ，实际上也是想找到距离触摸点最近的 `UIView` 。寻找最适合的 `UIView` 过程是从每个 `UIView` 的最顶层往最底层去找的，这和 `UIResponder` 响应链的过程不同，事件响应是 `UIEvent` 中的每个 `UITouch` 所属的 `UIView` 都确定后才开始的。

↓↓↓↓↓

以上就是触摸事件的基本流程，也就是找到了最合适的 `UITouch` 进行接收 `UIEvent` 。那 `UITouch` 如何找到合适的 `gestureRecognizers` 呢？系统会围绕 `UITouch` 所属的 `UIView` 及其祖先 `UIView` 的 `gesturerecognizer` 来确定一个 `UITouch` 的最佳  `gesturerecognizer` 。再次说明，从这一步开始的之前步骤，都是在找用户触摸点的最佳响应视图，也就是找到了用户触摸的视图是那个，但是并没法响应用户进行触摸操作，比如点击、双击、滑动等等，这就需要让 `UITouch` 再进行寻找最佳的 `UIResponder` 对象进行处理。注意，在该例子中，我并未对各个视图添加手势识别器，如果我们对各个视图添加了手势识别器，因为手势识别器比 `UIResponder` 具有更高的事件响应优先级，所以将会先把该事件传递给手势识别器，举个例子：在一个 `view` 上添加了 `tap` 手势识别器，并且还添加了一个 `tableView` ，此时当我们点击 `cell` 时是不会进行任何操作的，因为 `view` 已经把触摸事件给截获了。如果手势识别器没有进行对该次事件进行接收，才会被选中的最佳 `hitTest View` 进行捕获。再提示一下：手势实际上分为了两种，其一为离散型手势，包括点按、轻扫手势，其二为连续性手势，除了离散型手势外剩下的手势全都是，比如旋转、放大等等

↓↓↓↓↓

`UITouch` 所属的 `UIView` 和 `gesturerecognizer` 收到对应的 `UITouch` 和 `UIEvent` 对象后，按照 `UITouch` 目前所处的状态进行分别调用 `touchBeng` 、`touchMoved` 、`touchEnded` 、`touchCancled` 四大方法中的一个，也就是开始了正式的事件响应。这四大响应事件会按照 `UIResponder` 响应链一直从下往上传递，直到某个 `UIResponder` 因为主动响应了触摸事件而结束，切断了响应链，如果这一条响应链找上去都没有对其进行响应的对象，那最终会被 `UIApplication` 对象直接杀掉。也就是说，如果我们在 `scrollView` 中放了 `UImageView` 和 `UIButton` ，从 `UIImageView` 出发的滑动事件不会被其本身响应，而是被传递到了 `scrollView` 中，但是如果我们是从 `UIButton` 出发的事件，`UIButton` 首先会进行拦截判断，如果是点击事件，自己就接收进行处理，如果不是，例如滑动事件，那就丢掉，让该事件继续沿着响应链向上找到 `scrollView` 进行处理。还有一个时期需要注意，如果我们开启了并发手势，也就是允许多个手势同时执行，各自对应的 `gesturerecognizer` 方法直接拦截跟自己手势相同的 `UITouch` 对象，剩余的 `UITouch` 对象 `gesturerecognizer` 处理不了就会让它跟着响应链继续往上走。需要注意的是，每个事件响应者必定是 `UIResponder` 对象，且通过这四大方法进行响应触摸事件，并且每个 `UIResponder` 对象都已经实现了这四个方法，不过并没有对这四个方法做任何处理，如果我们需要做一些额外的操作，需要重写这四大方法。

↓↓↓↓↓

到这个环节，`UITouch` 对象的所有相关事件通通流动完毕，`CPU` 继续进行等待状态。

总结一下，触摸发生时，系统内核生成触摸事件，先由 `IOKit` 处理封装成 `IOHIDEvent` 对象，通过 `IPC` 传递给系统进程 `SpringBoard` ，而后再传递给前台 `APP` 处理。事件传递到 `APP` 内部时被封装成开发者可见的 `UIEvent` 对象，先经过 `hit-testing` 寻找第一响应者，而后由 `Window` 对象将事件传递给 `hit-tested view` ，并开始在响应链上的传递。一句话说就是从用户触摸到屏幕再到 `app` 做出处理，大概要经过三个阶段，系统处理阶段 → `SpringBoard.app` 处理阶段 → `App` 内部处理阶段，在开发中几乎我们也只在第三阶段搞事情。而且还要注意，在 `iOS` 中不是任何对象都能够去处理对象，而是只有继承了 `UIResponder` 的对象才能接收并处理事件，也就是我们所说的响应者对象，比如我们常见的： `UIApplication` 、`UIViewController` 、`UIView` 等等

