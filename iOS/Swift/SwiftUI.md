# SwiftUI 注意点

## 前言
`SwiftUI` 让我找到了初学 iOS 开发时的乐趣。强烈推荐：[SWIFTUI BY EXAMPLE](https://www.hackingwithswift.com/quick-start/swiftui)

### Live Mode 情况下只是展示 UI，逻辑实际上并不能跑

### `some View` 这一行代码是什么意思？
`Swift 5.1` 中新增了一个**不透明结果类型**。在 `SwiftUI` 中，同样也都是几乎所有的系统组件都继承自 `View` 这个**协议**，但 `View` 本身却带有个 `associatedtype` 关联类型，带有关联类型的协议不能作为**类型**来使用，返回一个 `View` 时必须指定该 `View` 的类型。

指定该 `View` 的类型有两种方法，调用与 `View` 相关的方法时，在调用前指定好对应的 `View`（利用泛型）；在 `View` 内部返回指定类型的其它 `View`，如 `Button`、`Text` 等。

但是每次指定 `View` 的类型需要依赖于我们每次使用 `View` 时都知道其真正的类型，这实际上是很困难的，需要多次修改，那么使用 `some` 关键字，使用该关键字后，编译器可以根据返回值类型推断得到具体类型。

同时这也是 `Swift 5.1` 引入的新特性 `opaque result type`，其有：

* 所有的条件分支只能返回一个特定类型，不同则会编译报错
* 方法使用者依旧无法知道类型，（使用方不透明）
* 编译器知情具体类型，因此可以使用类型推断。


### 加入现有的 `UIViewController`
```swift
struct ViewControllerWrapper: UIViewControllerRepresentable {

    typealias UIViewControllerType = ViewController


    func makeUIViewController(context: UIViewControllerRepresentableContext<ViewControllerWrapper>) -> ViewControllerWrapper.UIViewControllerType {
        return ViewController()
    }

    func updateUIViewController(_ uiViewController: ViewControllerWrapper.UIViewControllerType, context: UIViewControllerRepresentableContext<ViewControllerWrapper>) {
        //
    }
}

struct MyView : View {
    var body: some View {
        ViewControllerWrapper()
    }
}
```

### `@State`、`@ObjcetBinding` 和 `@EnvironmentObject` 的区别
* 对于不变的常量直接传递给 SwiftUI 即可。
* 对于控件上需要管理的状态使用 @State 管理。
* 对于外部的事件变化使用 BindableObject 发送通知。
* 对于需要共享的视图可变数据使用 @ObjectBinding 管理。
* 不要出现多个状态同步管理，使用 @Binding 共享一个 Source of truth。
* 对于系统环境使用 @Enviroment 管理。
* 对于需要共享的不可变数据使用 @EnviromemntObject 管理。
* @Binding 具有引用语义，可以很好的和 @Binding @objectBinding @State 协作，避免出现多个数据不同步。

### SwiftUI 如何进行渲染子元素
1. 父视图为子视图提供预估尺寸
2. 子视图计算自己的实际尺寸
3. 父视图根据子视图的尺寸将子视图放在自身的坐标系中

实际上在写 `SwiftUI` 的代码，并不是在声明/创建和一个对象，就算是写了上百行，你都没有创建出任何一个 UI 对象，你一直都是在写 DSL，一直在写布局约束。

有时候任何的 `padding` 都没有写，却发现元素和元素之间实际上是有一点点间距的，这是因为 Apple 针对自家的人机交互指南自动填充的。

### 到底怎么样才是正确的 `SwiftUI` 开发模式
首先，需要确定的是 `SwiftUI` 提供了很多数据监听的方案，我们不再需要像之前那般手动同步“数据至视图”和“视图到数据”这两个环节，统统都可以交由 `Combine` 去处理。

那也就是说，之前 `ViewController` 中负责处理这两个环节的代码统统都没了，但是这不是说 `UIViewController` 没了，按照之前写 `Vue` 经验，做法是这样的：

在父组件（相当于是 `UIViewController`）中的 `created` 方法中发起网络请求。
    - 在「发起请求」到「元素渲染」这一环节之间是有时间差的。
    - 为了提供一个良好的用户体验，需要在这一环节中涉及到的变量做「默认值」处理，例如，列表变量要先给空之类。

换句话说，发起网络请求的时机可以是「元素渲染」之前或之后，但是渲染什么元素以及元素上的内容是是什么，这与网络请求无关，也就是说，我们在写 UI 时，要按照网络请求失败或网络请求数据为空来做。

#### 如果不需要依赖视图的创建，要怎么做？
通过 `BindableObject` 方式创建出数据中心，然后在创建视图组件的时候，创建该 `BindableObject` 对象，该对象在创建时去调用网络请求方法，网络请求不管再怎么快，都会比创建一个对象要慢得多。

所有在网络请求数据还未回来之前，UI 组件显示的内容为没有数据的内容，网络请求数据回来后再进行渲染。

### Redux
* 适用的场景
    * 多交互、多数据源
* 从组件的角度去看
    * 某个组件的状态，需要共享
    * 某个状态需要在任何地方都可以拿到
    * 一个组件需要改变全局状态
    * 一个组件需要改变另一个组件的状态
* Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。
* State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。
* View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。
* Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。
* Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。
* 由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象。
* 最好把 State 对象设成只读。你没法改变它，要得到新的 State，唯一办法就是生成一个新对象。这样的好处是，任何时候，与某个 View 对应的 State 总是一个不变的对象。

### 在使用 `HStack` 时如何使用设置两个元素左右布局

```swift
HStack(alignment: .center) {
    Image("5")
        .resizable()
        .frame(width: 50.0, height: 50.0)
    
    // 重点
    Spacer()
    
    Button(action: {
        
    }) {
        Image(systemName: "paperplane.fill")
            .imageScale(.large)
            .foregroundColor(.primary)
    }
}
```

### 把 `SwiftUI` 页面嵌入到 `UIKit` 页面中
* `UIViewController` -> `UIHostingController`
* `UIView` -> `UIHostingView`


### 在 `SwiftUI` 中如何设置代理
因为 `SwiftUI` 中没有 `TextView`，如果我们非要一个 `TextView` 只能从 `UIKit` 中「嫁接」一个包装过的 `TextView`过去，而不创建一个 `UITextView`。

经过一番操作后，把 `TextView` 创建出来了，但是发现需要获取一些例如「开始编辑」、「正在编辑」的状态，这个时候就需要 `Coordinator` 去协助了，

```swift
struct MASTextView: UIViewRepresentable {
    let now = Date()
    
    var isBeginEditng = false
    var nowTimeString: String {
        get {
            let dformatter = DateFormatter()
            dformatter.dateFormat = "yyyy年MM月dd日 HH:mm"
            return dformatter.string(from: now)
        }
    }
    
    // 显示声明协调器
    func makeCoordinator() -> MASTextView.Coordinator {
        Coordinator(self)
    }
    
    func makeUIView(context: Context) -> UITextView {
        let tv = UITextView()
        tv.tintColor = .black
        tv.font = UIFont.systemFont(ofSize: 18)
        tv.delegate = context.coordinator
        
        tv.text = "在 \(nowTimeString) 写下"
        return tv
    }

    func updateUIView(_ uiView: UITextView, context: Context) {
        
    }
    
    class Coordinator: NSObject, UITextViewDelegate {
        var textView: MASTextView
        
        init(_ textView: MASTextView) {
            self.textView = textView
        }
        
        func textViewShouldBeginEditing(_ textView: UITextView) -> Bool {
            if !self.textView.isBeginEditng {
                self.textView.isBeginEditng = true
                textView.text = ""
            }
            return true
        }
    }
}
```

### 设置 `BindableObject` 时的一些问题
```swift
class AritcleManager: BindableObject {
    var willChange = PassthroughSubject<Void, Never>()
    
    var article = [Article]() {
        willSet {
            willChange.send(())
        }
    }
}
```

上述代码中 `PassthroughSubject` 里的 `Void` 和 `Never` 是什么意思？
* `Void`。指明我们要在数据改变时传递什么值，因为在例如用 `BindableObject` 来实现用户管理类，在用户管理类中又有用户的 `viewModel` 和 `token`，当用户退出登录时，`token` 清空，`viewModel` 也要清空；用户再次登录时，`token` 被赋值，`viewModel` 也拿到了新用户的信息数据，此时只需要监听 `token` 的变化，并把 `viewModel` 发布出去即可。
* `Never`。指明我们是否要连带错误类型也通知出去。`Never` 表示通知时什么错误类型也不带上，可以按照需求定义为 `NetworkError`。

### 千万注意代码是有顺序的
如果我们想要给一个 `View` 添加触摸事件，会下意识的按照 `UIKit` 的做法去做，可能设置这个 `View` 的 `frame` 属性，也可能先给这个 `View` 这个添加触摸手势，在 `UIKit` 中代码的先后顺序显得不是那么重要。

但在 `SwiftUI` 中就非常重要了，必须先设置好 `View` 的 `frame` 才能添加成功触摸手势，要不会失效。

### 当遇到单个 `View` 无法撑起整个布局
善用 `Spacer()`

### SwiftUI 的重新预览问题
* Xcode 的预览使用了动态替换 `body` 属性的特性，但是它有一些局限，当 `body` 以外的部分被改变时，将导致 `ContentView` 需要整个重新编译时，比如在 `body` 之外添加一个存储属性 `var a = 2333`，必须再次点击 Resume 按钮才能重新开始预览。

### 圆角设置
`cornerRadius` 通过包装的方式为 `View` 添加圆角，并返回**新的 `View`**。

### `ForEach` 遍历怎么做到不需要写 `id:\.balabala`
* `ForEach` 是个 `DynamicViewContent` 类型，所以在实现 `List` 的侧滑删除时，需要把内容到 `ForEach` 中进行处理，因为 `onDelete` 事件要求 `DynamicViewContent` 类型。
* `ForEach` 用来列举元素，并生成对应的 view collection 类型，🉑️一个数组，且数组中的元素需要满足 `Identifiable` 协议。如果数组元素不满足 `Identifiable` 协议，需要使用 `ForEach(_:id:)` 来通过某个支持 `Hashable` 的 key path 获取一个等效的元素是 `Identifiable` 的数组。

### `@State` 的一些细节
和一般的存储属性不同，`@State` 修饰的值，在 SwiftUI 内部会被自动转换为一对 setter 和 getter，对这个属性进行赋值的操作将会触发 `View` 的刷新，它的 `body` 会再次被调用，底层渲染引擎会找出洁面上与这个值相关的的更改部分，并进行刷新。
### 为什么不能对各个层级的组件都使用 `@State` ？
* `@State` 仅能在属性本身被设置时触发 UI 刷新，所以一般会直接拿一个值类型变量用于记录状态；
* 但值类型的变量在多个组件层级之间进行传递时，该值将会遵循值语义发生复制，而不是引用。导致每个组件的状态值都不一样。
* `@Binding` 就是用来解决这个问题的。所做的事情时将值语义属性「转换」为引用语义。
    * 对 `@Binding` 的属性进行赋值，改变的将不是属性本身，而是它的引用，这个改变将被向外传递。
    * 所以是，每个组件如果需要被其它组件里的值影响，都需要使用 `@Bingding` 去修饰。

### 在传递属性的时候，在前面加上一个美元符号 `$`。
在 Swift5 中，在一个 `@` 符号修饰的属性前几上 `$` 所取得的值，称之为**投影属性**（projecttion property）。
    * 不是所有的 `@`  属性都有提供 `$` 的投影访问方式。
    * `$var` 将 `State` 转换成了引用语义的 `Binding`，并向下传递。

### `@` 属性在 Swift 中的正式名称是属性包装（property Wrapper）。

### 使用 `ObservableObject` 有两种写法：
```swift
let objectWillChange = PassthroughSubject<Void, Never>() 
var obj: objModel = ObjModel() {
        willSet { objectWillChange.send() 
    } 
} 
```

```swift
@Published var brain: CalculatorBrain = .left("0") 
```

### 多个 State 发送改变时，SwiftUI 要怎么变化？
我的猜测：在当前 `RunLoop` 中，收到多个 State 的改变，在该 `RunLoop` 结束时用最后一个 State 进行计算。

### `Publisher` 可以发布的事件类型
* 类型为 Output 的新值:这代表事件流中出现了新的值。
* 类型为 Failure 的错误:这代表事件流中发生了问题，事件流到此终止。 
* 完成事件:表示事件流中所有的元素都已经发布结束，事件流到此终止。

我们将最终会终结的事件流称为**有限事件流**，而将不会发出 failure 或者 finished 的事件流 称为**无限事件流**。

Publisher 的结束事件有两种可能:代表正常完成的 `.finished` 和代表发生了某个错误的 `.failure`，两者都表示 Publisher 不再会有新的事件发出。

### 多个 `Publisher` 怎么办？
通过一系列组合，我们可以得到一个响应式的 `Publisher` 链条:当链条最上端的 `Publisher` 发布某个事件后，链条中的各个 `Operator` 对事件和数据进行处理。在链条的末端我们希望最终能得到可以直接驱动 UI 状态的事件和数据。

### `sink`
可以通过 `sink` 订阅 Publisher 事件。Subscriber 可以指定想要接收的新值的个数，这不仅在订阅初期可以通过 `Subscription.request` 来告知 Publisher，也可以通过 `Subscriber.receive` 返回特定的 `Subscribers.Demand` 值来指定接下来能够处理的值的个数。通 过这些机制，Combine 将可以实现背压 (Backpressure)。`.unlimited` 表示不设上限，但当上游 Publisher 的值生产速度大于下游的消费速度时，下游的缓冲区就会发生溢出。指定合适的背压策略，通过控制上限，可以让系统下游不发生崩溃的同时，有机会对部分溢出事件做额外处理 (比如丢弃或者告 知上游不要再接受新的事件)。

在客户端开发中，需要处理背压的场景非常有限，但是在服务端开发处理大 规模数据时，这会是无法绕过机制。

### `assign`
通过 `assign` 绑定 Publisher 值。Combine 里还有另一个内建的 Subscriber: `Subscribers.Assign`，它可以用来将 Publisher 的输出值通过 key path 绑定到一个 对象的属性上去。

* 注意 assign 所 接受的第一个参数的类型为 ReferenceWritableKeyPath，也就是说，只有 class 上 用 var 声明的属性可以通过 assign 来直接赋值。
* assign 的另一个 “限制” 是，上游 Publisher 的 `Failure` 的类型必须是 `Never`。如果 上游 Publisher 可能会发生错误，我们则必须先对它进行处理，比如使用 `replaceError` 或者 `catch` 来把错误在绑定之前就 “消化” 掉。


### Subject
`sink` 提供了由函数响应式向指令式编程转变的露肩的话，`Subject` 则补全了这条通路的另一侧：它让你可以将传统的指令式异步 API 里的事件和信号转换到响应式的世界中去。

Combine 中内置提供了两种常用的 `Subject` 类型：`PassthroughSubject` 和 `CurrentValueSubject`。

* `PassthroughSubject` 简单地将通过 `send` 接收到的事件转 发给下游的其他 Publisher 或 Subscriber。

* `CurrentValueSubject` 则会包装和持有一个值，并在 设置该值时发送事件并保留新的值。

### Scheduler
如果说 `Publisher` 决定了发布怎样的 (what) 事件流的话，`Scheduler` 所要解决的就 是两个问题：在什么地方 (where)，以及在什么时候 (when) 来发布事件和执行代码。

* Combine 里提供了 `receive(on:options:) ` 来让下游在指定的线程中接收事件。
* `RunLoop` 就是一个实现了 `Scheduler` 协议的类型，它知道要如何执行后续的订阅任务。
* 比较常见的两种操作是 `delay` 和 `debounce`。`delay` 简单地将所有事件按照一定事件 延后。`debounce` 则是设置了一个计时器，在事件第一次到来时，计时器启动。在计 时器有效期间，每次接收到新值，则将计时器时间重置。当且仅当计时窗口中没有新 的值到来时，最后一次事件的值才会被当作新的事件发送出去。主要的一个运用场景：用户键入内容时，实时地给出搜索结果，可使用 `debounce` 进行 1s 的延时操作。
* 它们都是 Publisher 上的扩展方 法，并返回一个新的 Publisher。

### `Operator`
在 `Publisher` 上也存在一个 `map` 函数，我们可以通过类似的方式，对 output 的元素进行变形:

```swift
check("Map") {
    // " 注意我们是在 `Publisher` 上调用了 `map`
    [1,2,3]
        .publisher
        .map{$0*2}
}
```

* 经过 `reduce` 变形后，新的 `Publisher` 只会在接到上游发出的 `finished` 事件后，才会将 `reduce` 后的结果发布出来。

### `scan`
类一边进行重复操作，一边将每一步中间状态发送出去的场景十 分普遍，因此 `Combine` 内置提供了 `scan` 这个 `Operator`。

`scan` 一个最常见的使用场景是在某个下载任务执行期间，接受 `URLSession` 的数据 回调，将接收到的数据量做累加来提供一个下载进度条的界面。

### 为 `Array` 标准库添加 `scan` 操作
有些情况下，除了最终的结果，我们也有可能会想要把中途的过程保存下来。在 `Array` 中，这种操作一般叫做 `scan`。这个方法在标准库中并不存在，不过我们可以很 容易地添加一个:

```swift
extension Sequence {
public func scan<ResultElement>(
_ initial: ResultElement,
_ nextPartialResult: (ResultElement, Element) !" ResultElement ) !" [ResultElement] {
var result: [ResultElement] = []
forxinself{
            result.append(nextPartialResult(result.last ?? initial, x))
        }
        return result
    } 
}
```

调用该方法的方式和 reduce 几乎相同: 

```swift
[1,2,3,4,5].scan(0, +)
// " [1, 3, 6, 10, 15]
```

### `compactMap`
它的作用是将 `map` 结果中那些 `nil` 的元素去除掉，这个操
作通常会 “压缩” 结果，让其中的元素数减少。

```swift
["1", "2", "3", "cat", "5"]
    .publisher
    .compactMap { Int($0) }
```

直接使用 Swift 进行函数式编程是这样的：

```swift
["1", "2", "3", "cat", "5"]
    .publisher
    .map { Int($0) } .filter { $0 !" nil } .map { $0! }
```

### `flatMap`
`flatMap` 的变形闭包里需要返回 一个 `Publisher`。也就是说，`flatMap` 将会涉及两个 `Publisher`:一个是 `flatMap` 操作本身所作用的外层 `Publisher`，一个是 `flatMap` 所接受的变形闭包中返回的内层 `Publisher`。flatMap 将外层 Publisher 发出的事件中的值传递给内层 `Publisher`，然 后汇总内层 `Publisher` 给出的事件输出，作为最终变形后的结果。

### `removeDuplicates`
```swift
["S", "Sw", "Sw", "Sw", "Swi","Swif", "Swift", "Swift", "Swif"]
    .publisher
    .removeDuplicates()
```

上例中，“Sw” 连续出现了三次，“Swift” 出现了两次，而经过移除操作后，我们得到 的是一系列没有重复的字符串事件。`removeDuplicates` 经常被用来减少那些非常消 耗资源的操作，比如由事件触发造成的网络请求或者图片渲染。如果当作为源头的 数据没有改变时，所预期得到的结果也不会变化的话，那么就没有必要去重复这样 操作。在源头将重复的事件移除，可以让下游的事件流也变得简单。

### 错误类型不一致转换
`map` 对 Output 进行转换，`mapError` 对 Failure 进行转换。

可以对各种 Operator 加上 `try`，如 `tryMap`，`tryReduce` 等，当你有需要在数据转换或者处理时，将事件流以错误进行终止，都可以使用对应操作的 `try` 版本来进行抛出，并在订阅者一侧接收到对应的错误事件。

### 错误替换
在 Combine 里，有一些 Operator 是专门帮助事件流从错误中恢复的，最简单的是 `replaceError`，它会把错误替换成一个给定的值，并且立即发送 `finisheds` 事件。

### `Just`
如果我们想要 publisher 在完成之前发出一个值的话，可以使用 `Just`，它表示一个单一的值，在被订阅后，这个值会被发送出去，紧接着是 `finished`。

### 使用 `merge` 整个事件流


### `zip`
`zip` 将从两个序列中取出 `index` 相同的元素，把它们组合为**多元组**，然后放到返回的序列中去:

```swift
zip([1, 2, 3, 4, 5], ["A", "B", "C", "D"])

// [(1, "A"), (2, "B"), (3, "C"), (4, "D")]
```

`zip` 在时序语义上更接近于 “当...且...”，当 Publisher1 发布值，且 Publisher2 发布值时，将两个值合并，作为新的事件发布出去。在实践中，`zip` 经常被用在合并多个 异步事件的结果，比如同时发出了多个网络请求，希望在它们全部完成的时候把结 果合并在一起。

### `combineLatest`
`combineLatest` 是一个很典型的例子，和 `zip` 相对，它的语义接近于 “当...或...”，当 Publisher1 发布 值，或者 Publisher2 发布值时，将两个值合并，作为新的事件发布出去。

combineLatest 被用来处理多个可变状态，在其中某一个状态发生变化时，获取这些全部状态的最新值。比如你的 UI 上有多个 `TextField`，你想要在其中某 一个值变动时获取到所有 `TextField` 中的值进行检查，例如在**用户注册**时。

### `Future`
`Future` 只能为我们提供一次性 Publisher:对于提供的 promise，你只 有两种选择:发送一个值并让 Publisher 正常结束，或者发送一个错误。因此， Future 只适用于那些必然会产生事件结果，且至多只会产生一个结果的场景。比如网络请求:它要么成功并返回数据及响应，要么直接失败并给出 `URLError`。一个 `dataTask` 的网络请求不会永远不发送任何事件，也不会产生多次的 响应，用 Future 进行包装恰得其所。如果你的异步 API 有可能不发送任何一个值，而是可能发布两个或更多的值的话，你会需要一个更加一般性的 Publisher 类型来把指令式程序转换为响应式程序。

### 对于多个 Subscriber 对应一个 Publisher 的情况
如果我们不想让订阅行为反复发 生 (比如上例中订阅时会发生网络请求)，而是想要共享这个 Publisher 的话，使用 share() 将它转变为引用类型的 class。

### `throttle`
它在收到一个事件后开始计时，并忽略计时周期内的后续输入。

### 每一个 `@State` 都是一个数据源

### * 当 `Text` 布局约束太小，以至于产生了异常的内容缩减
例如缩减了尾部文字，但我们却想要缩减头部文字，此时可以使用 `View` 的 `layoutPriority(1)` 方法从默认的 0 更改为 1。

### 如果想要 `Image` 和 `Text` 进行同一个基线进行对齐
* 可以对 `Image` 使用 `.alighnmentGuide` 自定义基线距离。
* 文本基线对齐的方法：`HStack(alignment: .lastTextBaseline) {}`

![自定义对齐](https://i.loli.net/2019/08/20/3TZrl7QOIXEcW8s.png)

![自定义对齐的使用](https://i.loli.net/2019/08/20/Efe61CyhBqK9AmH.png)


### 图形绘制
当需要绘制大量的内容时，比如一组图形或一组文字，可以使用 `.drawingGroup` 进行，开启后，将会把绘制任务丢到 `Metal` 里使用 GPU 进行绘制加速。

![图形绘制](https://i.loli.net/2019/08/20/vJVx9UGD3l2qkZN.png)

