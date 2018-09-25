# Swift

这篇文章主要记录我在学习Swift的一些记录、Swift是14年的WWDC上苹果推出的一门新语言，这是一门非常新的语言，而且在不停的发展当中，对新手非常的友好，可以断定的是Swift将来一定是苹果推的主流开发语言。Objective-C在五年不会消亡，因为OC的强大不是Swift能一时半会取代的，或者说将来会大量存在由OC和Swift混编的项目吧（虽然说现在也有，但还是比较少，估计以后都会这样了）。

以下内容为 Swift 学习过程中需要注意的地方，当前版本为 4.2 ，来源于 [iOSCaff](https://ioscaff.com) 社区，欢迎大家一同来玩耍。

## Swift 4.2 概览

* Swift 的不需要为了输入/输出或者字符串处理而去导入一个单独的库；
* 全局作用域中的代码会自动作为程序的入口，所以你并不需要 `main()` 函数；
* 不需要在句尾写分号；

### 简单值
* `let` 声明常量，`var` 声明变量。常量在编译时不需要赋初值，但后续只能对它赋值一次。
* 你不用明确的声明类型，因为编译器会根据你所创建的常量或变量来推断它们的类型。
* 对于占用多行的字符串可使用三个引号 `"""` ，如：
    ```Swift
    let quotation = """
    I said "I have \(apples) apples."
    And then I said "I have \(apples + oranges) pieces of fruit."
    """
    ```
* 使用 `[]` 创建空数组，使用 `[:]` 创建空字典。

### 控制流
* 在 `if` 语句中，条件语句必须是布尔表达式，所以 `if score {...}` 等类似的代码将会报错，而且不会与 0 做隐式的比较，也就是说在 `OC` 中经常用于判断一个变量是否存在的写法在 `Swift` 中要使用 `if` 和 `let` 来结合处理值缺失的情况，如下所示：
    ```Swift
    if let s = score {
        printf(s)
    }
    ```
* 处理可选值还可以使用 `??` 加入 **默认值** 的做法，如下所示：
    ```Swift
    let score: String? = nil
    let myScore = score ?? "zore"
    ```
* 运行完 `switch` 语句中与 `case` 匹配的代码后，程序会直接从 `switch` 语句退出，下一个 `case` 语句不会被执行；

### 函数和闭包
* 使用元组来生成复合值，例如使用元组来让一个函数返回多个值。该元组的元素可以通过名称或数字还获取，例如：
    ```Swift
    func calculateStatistics(scores: [Int]) -> (min: Int, max: Int, sum: Int) {
        var min = scores[0]
        var max = scores[0]
        var sum = 0

        for score in scores {
            if score > max {
                max = score
            } else if score < min {
                min = score
            }
            sum += score
        }

        return (min, max, sum)
    }
    let statistics = calculateStatistics(scores: [5, 3, 100, 3, 9])
    print(statistics.sum)
    print(statistics.2)
    ```
* 函数其实是一种特殊的闭包，它是可以延后执行的一段代码。在闭包里的代码可以访问到闭包作用域范围内的变量和函数（即使是在不同的作用域执行）。可以通过使用 `{}` 来创建一个 **匿名闭包** ，使用 `in` 将参数和返回值类型与闭包函数体分离，如下所示：
    ```Swift
    numbers.map({ (number: Int) -> Int in
        let result = 3 * number
        return result
    })
    ```
* 当我们一直一个闭包的类型，比如作为一个代理的回调，可以完全忽略参赛、返回值，甚至两个都忽略，单个语句闭包会把它的语句值当做结果返回，如下所示：
    ```Swift
    let mappedNumbers = numbers.map({ number in 3 * number })
    print(mappedNumbers)
    ```
* 还可以通过参数位置而不是参赛名字来引用参数（在短闭包方法中非常有用）。当一个闭包作为最后一个参数传给一个函数时，它可以 **直接跟在括号后面** 。当一个闭包是传给函数的唯一参数时，则可以 **完全忽略括号** ：
    ```Swift
    let sortedNumbers = numbers.sorted { $0 > $1 }
    print(sortedNumbers)
    ```

### 协议和拓展
* 与 `OC` 不同的是，结构体和枚举可以拥有方法，其中方法也可以为实例方法，可以为类方法。虽然结构体和枚举可以定义自己的方法，但是默认情况下，实例方法中是不能修改值类型的属性的，为了能够在实例方法中修改属性值，可以在方法定义前添加关键字 `mutating` ，如下所示：
    ```Swift
    struct Point {
    var x = 0, y = 0
    
    mutating func moveXBy(x:Int,yBy y:Int) {
        self.x += x
        self.y += y
        }
    }
    
    var p = Point(x: 5, y: 5)
    p.moveXBy(3, yBy: 3)
    ```

* 使用 `extension` 可以为现有类型添加功能，例如新方法和计算属性。可以使用拓展将协议一致性添加到其它地方声明的类型，甚至是从其它库或者框架导入的类型：
    ```Swift
    extension Int: ExampleProtocol {
    var simpleDescription: String {
        return "The number \(self)"
    }
        mutating func adjust() {
            self += 42
        }
    }
    print(7.simpleDescription)
    ```

### 错误处理
* 使用 `defer` 来处理函数执行完毕后需要处理的事情，无论这个函数是否抛出异常，该部分代码都会被执行。
    ```Swift
    func fridgeContains(_ food: String) -> Bool {
        fridgeIsOpen = true
        defer {
            fridgeIsOpen = false
        }

        let result = fridgeContent.contains(food)
        return result
    }
    ```

### 泛型
* 在类型名称后紧接 `where` 来明确表示一系列需求——例如，要求类型实现一个协议，要求两个类型必须相同，或者要求类必须继承来自特定的父类。
    ```Swift
    func anyCommonElements<T: Sequence, U: Sequence>(_ lhs: T, _ rhs: U) -> Bool
    where T.Iterator.Element: Equatable, T.Iterator.Element == U.Iterator.Element {
        for lhsItem in lhs {
            for rhsItem in rhs {
                if lhsItem == rhsItem {
                    return true
                }
            }
        }
        return false
    }
    anyCommonElements([1, 2, 3], [3])
    ```



1. **mutating**：为了能够在struct和enume中修改方法中修改属性值，可以在方法定义前添加关键字。详见：[https://blog.csdn.net/jeffasd/article/details/55104351](https://blog.csdn.net/jeffasd/article/details/55104351)

2. **@autoclosure**：主要是为了化简闭包嵌套写法，详见：[https://www.jianshu.com/p/99ade4feb8c1](https://www.jianshu.com/p/99ade4feb8c1)

3. Swift注释中的注释写法真的是不停的刷新我的三观。😂。。。详见:[https://blog.csdn.net/ruglcc/article/details/53007850](https://blog.csdn.net/ruglcc/article/details/53007850)和[https://www.jianshu.com/p/eda4a8dc0b3f](https://www.jianshu.com/p/eda4a8dc0b3f)

4. **typealias**：定义别名，我觉得它最好用的地方在于给闭包使用，详见：[https://www.jianshu.com/p/082202b9dc17](https://www.jianshu.com/p/082202b9dc17)

5. **Equatable**：自定义相等协议。不过我觉得貌似也能通过一个方法去搞定？不管怎么说，通过`==`符号更加直观吧，关于使用它的有点和缺点，详见：[https://www.natashatherobot.com/implementing-equatable-for-protocols-swift/](https://www.natashatherobot.com/implementing-equatable-for-protocols-swift/)

6. `Array<[String : String]>`的取key和value问题。

 背景：最近在上编译原理的课，第一个实验为编写一个词法分析器，原本我的想法是使用Qt来完成，写C/C++。但是突然一闪，哈！这是使用Swift进行开发的好时机！因此，我就开始了......省略五千字，详细见[这篇文章](../macOS/macOS开发（词法分析器）.md)

 在开发词法分析器的过程中，我遇到了这么个问题，知道词法分析的同学都能明白，在词法分析阶段需要一个tokens二元组接收词法分析后的结果，因此我还是用了OC中的那套思想，可以用一个`NSMutableArray`，其中装一个个的字典，其实也有相关放的是弄个二维数组，但是觉得套个字典数组可能比较好吧。

 但也就是这个“可能比较好吧”的想法，直接导致了这么个坑的出现，发现不管怎么搞，都没法使用自己之前已掌握的能力去筛选出数组中每个字典的key和value。搞到最后，发现如果各位同学也是跟我一样，有`Array<String : String>`这个格式数据的需求，想要遍历出数组中每个元素的key和value，那么就可以参考下边这种做法：

 ```Swift
 // 给NSTextField刷新上key，row为当前数组的下标，先获取到所有keys，然后取第一个，虽然实际上只有一个
 textField.stringValue = Array(tokenArray[row].keys)[0]

 //给NSTextField刷新上value
 textField.stringValue = Array(tokenArray[row].values)[0]

 ```

7. Swift中的！和？（解包实在是太恶心的一件事了）[http://www.jb51.net/article/100382.htm](http://www.jb51.net/article/100382.htm)，感觉这篇文章是抄的，但是写的内容还算明了。

8. Swift中的值类型和引用类型。[https://www.jianshu.com/p/ba12b64f6350](https://www.jianshu.com/p/ba12b64f6350)

9. Swift中的属性相关。神奇的set和get。[https://www.jianshu.com/p/071024b38a8b](https://www.jianshu.com/p/071024b38a8b)

10. **继承**。对于自定义的类而言，Objective-C的类，不能继承自Swift的类，即要混编的OC类不能是Swift类的子类。反过来，需要混编的Swift类可以继承自OC的类。

11. Swift中的方法选择器#selector还是用到了OC的runtime。😔，还是不够Swifty。

12. 混编项目中，如果你的协议是用的Swift写的，而且其中有`option`方法，那就要在对应的方法前面加上`@ObjC`关键词。

13. 如何用给当前`WKWebView`的request添加`Cookie`?
    首先是`setCookie`方法，
    ```Swift
    private func setCookie() {
        var ticketCookieProperties = [AnyHashable: Any]()
        ticketCookieProperties[HTTPCookiePropertyKey.domain]    = "Your hostname"
        ticketCookieProperties[HTTPCookiePropertyKey.name]      = "Your Cookie.name"
        ticketCookieProperties[HTTPCookiePropertyKey.value]     = "Your Cookie.value"
        ticketCookieProperties[HTTPCookiePropertyKey.path]      = "/"
        ticketCookieProperties[HTTPCookiePropertyKey.expires]   = Date().addingTimeInterval(3600)
        let ticketCookie = HTTPCookie.init(properties: ticketCookieProperties as! [HTTPCookiePropertyKey : Any] )
        HTTPCookieStorage.shared.setCookie(ticketCookie!)
        
        var usernameCookieProperties = [AnyHashable: Any]()
        usernameCookieProperties[HTTPCookiePropertyKey.domain]    = "Your hostname"
        usernameCookieProperties[HTTPCookiePropertyKey.name]      = "Your Cookie.name"
        usernameCookieProperties[HTTPCookiePropertyKey.value]     = "Your Cookie.value"
        usernameCookieProperties[HTTPCookiePropertyKey.path]      = "/"
        usernameCookieProperties[HTTPCookiePropertyKey.expires]   = Date().addingTimeInterval(3600)
        let usernameCookie = HTTPCookie.init(properties: usernameCookieProperties as! [HTTPCookiePropertyKey : Any] )
        HTTPCookieStorage.shared.setCookie(usernameCookie!)
    }
    ```

    接着是`readCurrentCookie`方法，把之前设置全局`Cookie`取出来，
    ```Swift
    private func readCurrentCookie() -> String {
        let cookieJar = HTTPCookieStorage.shared
        var cookieString = ""
        for cookie: HTTPCookie in cookieJar.cookies! as Array {
            cookieString = cookieString + "\(cookie.name)=\(cookie.value);"
        }
        return cookieString
    }
    ```

    最后是给当前的`WKWebView`的request添加`Cookie`，
    ```Swift
    var request = URLRequest.init(url: URL(string: requestURL!)!)
    // 注入公网所需Cookie
    request.addValue(readCurrentCookie(), forHTTPHeaderField: "Cookie")
    webView?.load(request)
    ```

    记得使用之前先调用`setCookie()`方法把我们需要的相关`Cookie`给种上。

14. 更换当前App的启动图时，如果运行App时未出现，应该删掉App重装一次，强制删除安装App的缓存。

15. 监听系统音量三部曲：
    ```Swift
        // 1
        import AVFoundation

        // 2
        try! AVAudioSession.sharedInstance().setActive(true)
        UIApplication.shared.beginReceivingRemoteControlEvents()

        NotificationCenter.default.addObserver(self, selector: #selector(volumeValueChange), name: NSNotification.Name(rawValue: "AVSystemController_SystemVolumeDidChangeNotification"), object: nil)

        // 3 
        // 在对应的方法中通过AVAudioSession来获取音量
        var tempVolume = AVAudioSession.sharedInstance().outputVolume
    ```

16. 想要隐藏系统音量界面，需要在对应VC的`viewDidLoad` or `viewWillAppear`中写下：
    ```Swift
    UIApplication.shared.keyWindow?.insertSubview(MPVolumeView(frame: CGRect.init(x: -2000, y: -2000,
                                                                                  width: 1, height: 1)), at: 0)
    ```
    当然，需求前提得是隐藏，如果你要想自定义，那就不要把该视图移除当前可视区域中，如果我们对该视图啥都不做，那就会只出现一条Slider，拖动该Slider即可调节音量。


17. 要善于使用 `guard` 和 `defer` 进行代码优化。 `guard` 做函数预处理，筛出不能进入函数执行代码块的情况；`defer` 用于处理函数执行完后的变量处理。

### 菱形问题和菱形继承（菱形，也称钻石）
* **菱形继承**：当一个子类继承多个父类时，多个父类最终继承了同一个类，这种情况称为“菱形继承”。会导致比如祖先类的初始化方法中有一个计数器，父类 A 继承了该祖先类，父类 B 也继承类该祖先类，子类 C 则继承了 A 和 B ，则该祖先类的计数器则被计数了两次，若菱形结构重叠，还会计数更多次。

在 C++ 中可以使用 **虚继承** 的方式解决，详情可看 [https://www.cnblogs.com/BeyondAnyTime/archive/2012/06/05/2537451.html](https://www.cnblogs.com/BeyondAnyTime/archive/2012/06/05/2537451.html)

* **菱形问题**：多个父类实现了同一方法，子类无法判断继承哪个父类的情况。`Java` 中可用 `interface` 的方式解决， `Swift` 中可用 `protocol` 的方式解决。