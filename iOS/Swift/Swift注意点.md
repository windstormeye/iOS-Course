# Swift

这篇文章主要记录我在学习Swift的一些记录、Swift是14年的WWDC上苹果推出的一门新语言，这是一门非常新的语言，而且在不停的发展当中，对新手非常的友好，可以断定的是Swift将来一定是苹果推的主流开发语言。Objective-C在五年不会消亡，因为OC的强大不是Swift能一时半会取代的，或者说将来会大量存在由OC和Swift混编的项目吧（虽然说现在也有，但还是比较少，估计以后都会这样了）。


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