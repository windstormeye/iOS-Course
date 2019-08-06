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
* @Binding 具有引用语义，可以很好的和 @Binding @objectBinding * * @State 协作，避免出现多个数据不同步。

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