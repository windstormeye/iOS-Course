# SwiftUI 注意点

## 前言
`SwiftUI` 让我找到了初学 iOS 开发时的乐趣。强烈推荐：[SWIFTUI BY EXAMPLE](https://www.hackingwithswift.com/quick-start/swiftui)

### Live Mode 情况下只是展示 UI，逻辑实际上并不能跑

### `some View` 这一行代码是什么意思？
在 `SwiftUI` 中，同样也都是几乎所有的系统组件都继承自 `View` 这个**协议**，但 `View` 本身却带有个 `associatedtype` 关联类型，带有关联类型的协议不能作为**类型**来使用，返回一个 `View` 时必须指定该 `View` 的类型。

指定该 `View` 的类型有两种方法，调用与 `View` 相关的方法时，在调用前指定好对应的 `View`（利用泛型）；在 `View` 内部返回指定类型的其它 `View`，如 `Button`、`Text` 等。

但是每次指定 `View` 的类型需要依赖于我们每次使用 `View` 时都知道其真正的类型，这实际上是很困难的，需要多次修改，那么使用 `some` 关键字，使用该关键字后，编译器可以根据返回值类型推断得到具体类型。

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