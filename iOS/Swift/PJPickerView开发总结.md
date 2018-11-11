# PJPickerView 开发总结
今天周日继续撸码，继续完成另一个组件，给之取名为——`PJPickerView`，别以为它真的只是个`View` 哦，为了让它看上去显得不是太“重”，从而取了这个名字，本质上是个 `UIViewController`，可能你会觉得有些奇怪，为什么一个组件要上 `UIViewController` 呢？刚开始我也不想这么玩，听我慢慢道来。

## UI
还是先来看 UI，

<img src="https://i.loli.net/2018/11/11/5be8174e1e7d1.png" width="70%"/>

UI 已经画得十分清楚了，就是要让我们分离出一个组件来，而且还是能够自定义数据源的。

## 思考
* 肯定要用到 `UIPickerView` 和 `UIDatePickerView` ，只不过需要在 `UIPickerView` 上自定义一下；
* 要处理好蒙版。如果这还像之前那般偷懒，直接把整个组件添加到当前控制器视图上，蒙版的显示区域只能是 `UINavigationBar` 下的区域，这样会少了头部遮罩，十分奇怪；如果是把组件添加到当前显示的 `UIWindow` 上，那么 `statusBar` 里的运营商、电量和时间等信息也不会被遮罩，而且会异常明显的被高亮出来，如果你感兴趣的话，可以尝试把一个黑色的 `UIView` 直接添加到当前 `UIWindow` 上。
* 因为是个组件，所以是肯定不能走代理回调的。第一，Apple 自家的各种系统组件基本上都走的代理回调，再多写几个代理给自己或者其它人调用估计得炸了；第二，这可是高大上的 `Swift`，怎么还能屈服于老土的 `Objective-C` 时代的各种回调呢？闭包是一定要闭的！


## 实践
### 自定义 UIPickerView
`UIPickerView` 的各种回调使用方式和流程与 `UITableView` 及其类似，同样需要继承 `UIPickerViewDelegate, UIPickerViewDataSource`，并实现以下几个方法即可：

```Swift
// MARK: - Delegate
func numberOfComponents(in pickerView: UIPickerView) -> Int {
    // 告诉 UIPickerView 有多少组
}

func pickerView(_ pickerView: UIPickerView,
                numberOfRowsInComponent component: Int) -> Int {
    // 告诉 UIPickerView 每组下有多少条数据，component 为组别
}

func pickerView(_ pickerView: UIPickerView,
                titleForRow row: Int,
                forComponent component: Int) -> String? {
    // 返回 UIPickerView 每组下每条数据需要显示的内容，只能是字符串，如果要自定义 View 走 `pickerView(_ pickerView: UIPickerView, viewForRow row: Int, forComponent component: Int, reusing view: UIView?) -> UIView` 这个方法
}

func pickerView(_ pickerView: UIPickerView,
                didSelectRow row: Int,
                inComponent component: Int) {
    // 拿到 UIPickerView 当前组别和条数，相当于 section 和 row，注意：如果用户什么都选，默认在第一条，但此时因为用户并未进行操作，所以该代理方法里写的内容不会被执行
}
```

只要按照对应代理方法所提供的作用填写代码即可，因为 `PJPickerView` 最多只做两组数据，所以直接拿了一个二维数组去做了数据源，当然，如果调用者非得塞下超过两列的内容也不是不行，但显示出来的效果就会畸变，目前我除了再自定义一个数据源模型替代二维字符串数组外没有更好的想法。

### 闭包回调
在之前很长的一段时间里，我非常喜欢用代理回调做组件间，甚至 vc 间的事件处理回调，可能因为当时觉得这是最简单的一种方式了吧，到今年这段时间强制性压迫自己且到 `Swift` 上，如果在 `Swift` 上还用 OC 那一套流程去写代理回调，出来的效果全是浓浓到 OC 味道，一点都不 `Swifty`。

所以，我采用如下方式来进行处理回调：

```Swift
// 声明一个中间闭包，作为后边逃逸闭包的引用
private var complationHandler: ((String) -> Void)?

// ...

// MARK: - Public
class func showPickerView(viewModel: ((_ model: inout PickerModel) -> Void)?, complationHandler: @escaping (String) -> Void) {
    let picker = PJPickerView()
    picker.viewModel = PickerModel()
    if viewModel != nil {
        viewModel!(&picker.viewModel!)
        picker.initView()
    }
        
    picker.complationHandler = complationHandler
    // 这是重点方法，后文讲解
    picker.showPicker()
}
```

因为涉及到许多变量，所以在此我用了一个结构体去做了承载：

```Swift
struct PickerModel {
    var pickerType: pickerType = .time
    var dataArray = [[String]]()
    var titleString = ""
}

enum pickerType {
    case time
    case custom
}
```

不想在外部调用初始化器对 `PJPickerView` 做初始化，采用了类方法供外部调用，且在类方法内部对 `viewModel` 做初始化，通过 `inout` 关键字修改其为可变参数传出给外部，这样就可以达到在外部对 `viewModel` 设置好相关参数后，在类内部直接使用即可。

最后使用 `@escaping` 关键字把跟随的闭包设置为了逃逸闭包，用之前声明的 `complationHandler` 对该逃逸闭包进行引用，供对应方法进行调用，调用方式所示：

```Swift
@objc fileprivate func okButtonTapped() {
   
    // ...

    // finalString 为 UIPickerView 选中的字符串，在 didSelectRow 方法进行设置
    if complationHandler != nil {
        complationHandler!(finalString)
    }
}
```

这样就完成了当对 `UIPickerView` 进行选择时可以回调给调用方，而调用方可以这么来进行调用：

```Swift
PJPickerView.showPickerView(viewModel: { (viewModel) in
    viewModel.titleString = "感情状态"
    viewModel.pickerType = .custom
    viewModel.dataArray = [["单身", "约会中", "已婚"]]
}) { [weak self] finalString in
    if let `self` = self {
        self.loveTextField.text = finalString
    }
}
```

以上的这种调用方式就是为内心中相对较为完美的调用方法了！🤓

### 蒙版
经过以上几个步骤后，我们基本上已经把 `UIPickerView` 的主体搭建完毕，接下来进行蒙版的设计。

如果此时我们把 `PJPickerView` 带上蒙版（实际就是个 `UIView`）直接添加到 `ViewController.view` 上，蒙版只会占据 `ViewController.view.frame` 的区域，如果当前的这个 `ViewController` 在 `UINavigationBar` 下，会导致头部区域无法被蒙版覆盖，所以是肯定不能直接添加到 `ViewController` 上的。

之前我的偷懒做法是直接把组件添加到当前 `topWindow` 上，这样就能够除了顶部状态栏上以外全覆盖了，但问题是如果我们就想把包括顶部状态栏也一起覆盖掉呢？此时直接用 `UIApplications` 里的 `UIWindow`，比如这么把最上层 `UIWindow` 拿出来：

```Objc
+ (UIWindow *)TopWindow {
    UIWindow * window = [[UIApplication sharedApplication].delegate window];
    if ([[UIApplication sharedApplication] windows].count > 1) {
        NSArray *windowsArray = [[UIApplication sharedApplication] windows];
        window = [windowsArray lastObject];
    }
    return window;
}
```

默认情况且我们不做其它任何修改，这样拿到的 `UIWindow` 的 `windowLevel` 是 `normal`，而我们的状态栏所在的 `UIWindow` 是 `statusBar` 级别， `UIWindowLevel` 的三种级别排序为：`normal` < `statusBar` < `alert`，所以这才会出现了如果我们直接把组件添加到当前 `UIWindow` 上蒙版并不能覆盖到顶部状态栏部分。

所以解决办法时，再造一个 `UIWindow.Level == .alert` 的 `UIWindow` 作为组件的容器，为了更好的让 `UIWindow` 对组件进行管理，此时也就引出了为什么 `PJPickerView` 底层是个 `UIViewController` 而不是 `UIView` 的原因：

```Swift
private func initView() {
    // 把当前 window 拿到
    mainWindow = windowFromLevel(level: .normal)
    pickerWindow = windowFromLevel(level: .alert)
    
    if pickerWindow == nil {
        pickerWindow = UIWindow(frame: UIScreen.main.bounds)
        pickerWindow?.windowLevel = .alert
        pickerWindow?.backgroundColor = .clear
    }
    pickerWindow?.rootViewController = self
    pickerWindow?.isUserInteractionEnabled = true

    // ...
}

func windowFromLevel(level: UIWindow.Level) -> UIWindow? {
    let windows = UIApplication.shared.windows
    for window in windows {
        if (level == window.windowLevel) {
            return window
        }
    }
    return nil
}

// show 方法
private func showPicker() {
    pickerWindow?.makeKeyAndVisible()

    // ...
}

// MARK: - Actions
    @objc fileprivate func dismissView() {
        UIView.animate(withDuration: 0.25, animations: {
            // ...
        }) { (finished) in
            if finished {
                UIView.animate(withDuration: 0.25, animations: {
                    self.pickerWindow?.isHidden = true
                    self.pickerWindow?.removeFromSuperview()
                    self.pickerWindow?.rootViewController = nil
                    self.pickerWindow = nil
                }, completion: { (finished) in
                    if finished {
                        self.mainWindow?.makeKeyAndVisible()
                    }
                })
            }
        }
    }

```

### 成果

<img src="https://i.loli.net/2018/11/11/5be83e60dbc91.png" width="60%"/>

<img src="https://i.loli.net/2018/11/11/5be83e9728cd3.png" width="60%"/>

<img src="https://i.loli.net/2018/11/11/5be83ebd45295.png" width="60%"/>

## 总结
在实现 `PJPickerView` 的过程中，第一场较为完整的学习和经历了以下事情：
·
* 自定义 `UIPickerView`；
* 简单的闭包回调的设计；
* 对蒙版的思考；

总的来说在实现的过程中自己主要是在反思“高内聚，低耦合”的指导，之前的做法都太简单粗暴，而且太过啰嗦，第一次较为完整的思考了整个流程，肯定还是有不足之处，等到后续功力慢慢增长再来对它好好修补一翻吧～

只放出了部分核心代码，不保证能够完全复现，只提供个思路～不管怎么说这周末的过的很开心，把手上的事情又往前推进了一大步！