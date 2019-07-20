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