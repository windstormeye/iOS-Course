## 第一天

### 字典和集合
一般的字典和集合都要求它们的 Key 都必须实现 Hashable 协议，Cocoa中的基本数据类型都满足这一点。

### 字符串
Swift 中的字符串为值类型，而不是 OC 中的引用类型。

#### 判断字符串是否由数字构成
```Swift
var str1 = "123ws"
// nil
Int(str1)

var str2 = "123"
// not nil 
Int(str2)
```

### Swift 的访问修饰符
`private`： 当前类内使用
`fileprivate`：当前文件内使用
`internal`：（默认访问级别）整个 module 里使用
`public`：可被任何地方使用，但除了 ·mudule 外不可以被继承和 override 
`open`：可被任何地方使用

### 如何检测一个链表中是否有环？
用 **快行指针** 的做法，一个指针在前，一个在后，两个指针的间隔一般为 2 ，循环终止条件为链表尾，如果有快指针和慢指针走到一起了，则该链表成环。

### 栈和队列的转化
用栈实现队列（腾讯一面）：
先写一个转换函数，把栈 A 的元素都 pop 到 栈 B 中，进队相当于给栈 A push ，出队之前先执行转换函数，然后再 pop 栈 B 元素。

用队列实现栈（腾讯一面）：
先写一个转换函数，这个函数把队列 A 中的除了队尾元素外的所有元素都入队到队列 B 中，那么经过这个转换函数转换之后队列 A 中剩下的就是栈顶元素。

再写一个函数，把队列 B 和队列 A 进行对调，此时因为队列 A 已经全部出队，所以队列 B 为空，队列 A 为少了之前的队尾元素队列


---

## 第二天
### 二叉树的遍历
因为二叉树本身是由递归定义的，从原理上讲，所有二叉树的题目都可以用递归来解。二叉树的遍历主要由 `BFS` （前中后序遍历）和 `DFS` （层级遍历）两种组成，需要注意的是，广度优先遍历需要用队列进行搭配

### 排序算法
动画和代码展示：[https://www.cnblogs.com/onepixel/articles/7674659.html](https://www.cnblogs.com/onepixel/articles/7674659.html)


## 第三天
### 几乎都是算法
比如 排序、动态规划、二分查找等等。慢慢总结吧

---

## 第四天
### inout 关键字
使用 `inout` 关键字可以修改传入参数的原始值，调用的时候需要在对应的参数前加上符号 `&` ，类似 `C/C++` 中的指针。


### protocol
在转 `Swift` 将近三四个月的过程中，我居然一点都没有感到在写 `protocol` 时代理对象居然不加 `weak` 关键字感到奇怪。

举个例子，之前我是这么粗暴的写 `protocol` ：
```Swift
protocol PjhubsDelegate {
    func pjhubsDeleagteFunction()
}

class Pjhubs: UIView {
    var viewDelegate: PjhubsDelegate?
}
```

就这么写了三四个月，看书看着看着才猛的发现，为啥我要把 `weak` 去掉，遂改成了以下代码：

```swift
weak var viewDelegate: PjhubsDelegate?
```

此时，`Xcode` 给我报了个错，

`'weak' must not be applied to non-class-bound 'PjhubsDelegate'; consider adding a protocol conformance that has a class bound`

也就是说：`weak` 只能能添加到非类绑定的 `PjhubsDelegate` 上，考虑给其添加上一个类绑定。根据提示，代码修改为：

```Swift
protocol PjhubsDelegate: class {
    func pjhubsDeleagteFunction()
}

class Pjhubs: UIView {
    weak var viewDelegate: PjhubsDelegate?
}
```

总的来说， `weak` 修饰引用类型，而上文中我所定义的 `protocol` 为值类型，所以 `Xcode` 报了错。如果不加 `weak` 修饰，则表明我们的 `protocol` 可为枚举、结构体所使用，所以当我们使用 `weak` 修饰了代理对象，那么就要求代理（协议）为 `class-only` （只类使用）


### copy-on-write
值类型在复制时，新对象和原对象实际上在内存中指向同一块区域，只有当新对象发生改变时（增加或删除一个对象），才会给新对象开辟新内存区域。


### 属性观察
最开始的时候，我直接在 `Swift` 中拿了 `OC` 的思想做了 `setter & getter`  ，但是当我想只要 `setter` 时，一定要把 `getter` 写上，就算 `getter` 什么也不做，只是返回一个存储属性的值而已。

后边理解到了 `Swift` 中的属性观察，即 `willSet & didSet` ，意思就是方法名所代表的意思。需要注意的是，在初始化器中对属性的设定，以及在 `willSet & didSet` 方法中对属性的再次设定，都会出发调用属性观察。


### @autoclosure
[http://swifter.tips/autoclosure/](http://swifter.tips/autoclosure/)

---

## 第五天
### 柯里化（Curring）
[http://swifter.tips/currying/](http://swifter.tips/currying/)。说实话，书中和喵神的这篇 blog 描述的代码都很简单，主要是这种神奇的写法根本没见过，书中的例子是这样的，要求写一个函数满足只传入一个整数参数，返回该整数 +2 的值，这很简单对吧，但是实际的要求是只写这么一个函数，然后满足 +2、+3、+4 等，其实我内心直接就冒出多态、模版、范型啦这些东西，但仔细一想，不对啊，只能有一个参数，瞬间缓过神来，柯里化牛逼啊。😂。
```Swift
func add(_ num: Int) -> (Int) -> Int {
    return { val in
        return num + val
    }
}

let number2 = add(2)
print(number2(2))
```

### 实现一个函数：求 0 ～ 100 （包括 0 和 100）中为偶数并且恰好是其他数字平方的数字

```Swift
(0...100).map { $0 * $0 }.filter { $0 % 2 == 0 }
```

Swift 函数式编程的一些资料：[https://www.jianshu.com/p/7233f140e6c3](https://www.jianshu.com/p/7233f140e6c3)


### ARC
ARC 和 Garbage Collection 的区别在于：Garbage Collection 在运行时管理内存，可以解决 retain cycle， 而 ARC 在编译时管理内存


### @property 关键字
在 OC 中基本数据类型的默认关键字是 `atomic`、`readwrite` 和 `assign`，普通属性的默认关键字为 `atomic`，`readwrite` 和 `strong`


### 关键字 automic 和 nonatomic
* automic ： 修饰的对象保证 setter 和 getter 的完整性，任何线程访问它都可以的哦大一个完整的初始化对象。正是因为要保证操作的完整性，所以速度较慢。automic 比 nonatomic 安全，但也不是绝对的线程安全，当多个线程同时调用 set 和 get 时，会导致获得的对象值不一样。想要获得线程绝对安全，使用 @synchronized （个人觉得 @synchronized 的做法也不好，这是 iOS 中最垃圾的锁哈哈）
* nonatomic ： 修饰的对象不保证 set 和 get 的完整性，所以当多个线程访问它时可能会返回未初始化的对象，故其速度会比 atomic 快，但线程也不是安全的。


### runloop 和 线程的关系
runloop 是每一个线程一直运行的一个对象，它主要用来负责响应需要处理的各种事件和消息。每一个线程都有且仅有一个 runloop 与之对应，没有线程，就没有  runloop 。在所有线程中，只有主线程的 runloop 是默认启动的，main 函数会设置一个 NSRunLoop 对象，而其它线程的 runLoop 默认是没有启动的，可以通过 `[NSRunLoop currentRunLoop]` 启动。


### code show
```objc
NSString *firstStr = @"helloworld";
NSString *secondStr = @"helloworld";

if (firstString == secondStr) {
    NSLog(@"Equal");
} else {
    NSLong(@"Not Equal");
}
```

最终将打印出 "Equal"，`==` 该符号是判断着两个指针是否指向同一个对象。上段代码尽管指向不同对象，但它们的值相同，iOS 编译器优化了内存分配，当两个指针指向两个值一样的 `NSString` 时，两者指向同一个内存地址。



## 后续内容已分门别类进行了归档。










