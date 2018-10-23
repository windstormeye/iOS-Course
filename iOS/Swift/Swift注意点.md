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
    ```Swift·
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

## 基础
### 类型注解
* 我们可以在一行中定义多个相同类型的变量，使用逗号来分割，在最后的变量名后面加上一个类型注解：
```Swift
var red, green, blue: Double
```

### 命名常量和变量
* 如果我们需要使用 Swift 预留关键字来命名常量或变量时，用反引号 `(``)` 包围关键字，但最好不要这么做。

### 打印常量与变量
* 可以通过 `print(_:separator:terminator:)` 来打印一个常量或变量当前的值，该方法默认会添加一个换行符来终止它打印的行，如果我们想要一个没有换行符的值，可以这么写 `print(someValue, terminator: "")` 。


## 基本运算符
### 算术运算符
* 与 C 以及 Objective-C 不同的是，在 Swift 中默认情况下算术运算符不允许值溢出。但我们能通过 Swift 的溢出符号加入值溢出的行为：
    符号 | 作用
    --- | ---
    &+  | 溢出加法
    &-  | 溢出减法
    &*  | 溢出乘法
    &/  | 溢出除法
    &%  | 溢出求模

### 空合运算符 ??
其中有个写法需要非常有趣，利用三元运算符进行解包
```Swift
let c = (a != nil ? a! : b)
```

这种写法相当于 `??` ：
```Swift
let c = a ?? b
```

### 单侧区间
如果想要从数组中遍历出从索引为 2 的下标到结尾的所有元素，可以这么写：
```Swift
for name in names[2...] {
    print(name)
}
```

如果想要从数组中遍历从开始至倒数第二个元素，可以这么写：
```Swift
for name in names[...2] {
    print(name)
}
```

## 数组
在 Swift 中把 OC 的 `NSArray` 和 `NSMutableArray` 都统一成了 `Array` ，看起来好像就一种数据结构，但实际上它的实现有三种：
* `ContiguousArray<Element>`：效率最高，元素分配在连续的内存上。如果元素的是值类型（zhan shang），则 Swift 会自动调用 Array 的这种实现；如果注重效率推荐声明这种类型，尤其当元素大量是类类型时。

* `Array<Element>`： 会自动桥接到 OC 的 `NSArray` 上，如果是值类型，则其性能与 `ContiguousArray` 无差别。
* `ArraySlice<Element>`：它不是一个新的数组，只是一个片段，在内存上与原数组享用同一区域。

关于数组的简单的操作：
```Swift
// 声明并初始化重复值
let nums = [Int](repeating: 0, count: 5)

// 对数组进行升序排序
nums.sort()

// 对数组进行降序排序
nums.sort(by: >)
```

用数组实现栈：
```Swift
class Stack {
    var stack: [Any]
    var isEmpty: Bool { return stack.isEmpty }
    var peek: Any? { return stack.last }
    
    init() {
        stack = [Any]()
    }
    
    func push(object: Any) {
        stack.append(object)
    }
    
    func pop() -> Any? {
        if !isEmpty {
            return stack.removeLast()
        } else {
            return nil
        }
    }
}
```

注意一个操作 `reserveCapacity()` 。它为原数组预留空间，防止数组在增加或删除元素时反复申请内存空间或是创建新数组，适合用于创建和 `removeAll()` 时进行调用。以上这段代码还引入了一个新的问题，`Any` 和 `AnyObject` 有什么区别？官方编程指南中指出：

> AnyObjct 可以代表任何 class 类型的实例
> Any 可以表示任意类型，甚至方法（func）类型

在 OC 中有个 `id` 类型，编译器不会对声明为 `id` 类型的变量进行类型检查，因为它可以表示任意类型。在 Cocoa 框架中很多地方都使用了 `id` 来进行如参数传递和方法返回的操作，这是 OC 动态特性的一种表现，但现在的 Swift 主要还是使用 Cocoa 框架进行 iOS app 开发，因此为了与 Cocoa 框架协作，将原来的 `id` 类型使用了一个可以表示任意 class 类型的 `AnyObject` 类型进行替代。

但 `id` 和 `AnyObject` 是有区别的。在 Swift 中编译器不仅不会对 `AnyObject` 进行实例的方法调用做检查，甚至会对 `AnyObject` 的所有方法调用都返回 `Opitional` 结果，因为这是符合 OC 理念的，但在 Swift 中却会很危险，应该先确定 `AnyObjct` 真正的类型并进行转换以后再进行调用，如下代码所示：

```Swift
func someMethod() -> AnyObject? {
    // ...

    // 返回一个 AnyObject?，等价于在 Objective-C 中返回一个 id
    return result
}

let anyObject: AnyObject? = SomeClass.someMethod()
if let someInstance = anyObject as? SomeRealClass {
    // ...
    // 这里我们拿到了具体 SomeRealClass 的实例

    someInstance.funcOfSomeRealClass()
}
```

所有的 class 都隐式的实现了 `AnyObject` 接口，这也就是为什么只适用于 class 类型的原因，但在 Swift 中所有的基本数据类型，比如 `Array` 、 `Dictionary` 在 OC 中为 class 的类型却通通都是 `struct` 类型，所以应该用 `Any` 类型进行表示，它除了能够表示 class 外，还可以表示 `struct` 和 `enum` 在内的所有类型。再来举个例子：

```Swift
let swiftInt: Int = 1
let swiftString: String = "miao"

var array: [AnyObject] = []
array.append(swiftInt)
array.append(swiftString)
```

在这段代码中，`swiftInt` 实际上的类型为 `NSNumber`，`swiftString` 实际的类型为 `NSString` ，因为在 Swift 和 Cocoa 中的这几个类型是可以自动转换，我们显式地声明了需要 `AnyObject` ，编译器认为我们需要的是 Cocoa 类型而非原生类型，帮我们进行了自动的转换。如果我们这么做：

```Swift
let swiftInt: Int = 1
let swiftString: String = "miao"

var array: [Any] = []
array.append(swiftInt)
array.append(swiftString)
array
```

这就拿到了 Swift 的原生数据类型 `Int` 和 `String`，值得一提的是如果我们只使用 Swift 类型而不转为 Cocoa 类型，性能将会得到提升，所以应该尽可能的使用原生类型。但不要在代码中出现太多次，如果 `Any` 和 `AnyObject` 在代码中出现了很多次，这说明设计上出了问题，可以通过**泛型**做改造。

## 字典和集合
字典和集合（专指 `HashSet`）经常被使用很重要的一点查找数据的时间复杂度为 **O(1)** 。字典和集合要求它们的 `Key` 都必须遵守 `Hashable` 协议，Cocoa 中的基本数据类型都满足这一点。自定义的 class 需要实现 `Hashable` 而又因为 `Hashable` 是对 `Equable` 的拓展，所以还要重载 `==` 操作符。

一些关于字典和集合的使用操作：
```Swift
let primeNums: Set = [3, 5, 7, 9]
let oddNums: Set = [1, 3, 5, 6]

/// 交集
/// Intersection()的操作不影响原集合，而 `formIntersection()` 则会已影响原集合。
print(primeNums.intersection(oddNums))
/// 并集
/// union()的操作不影响原集合，而formUnion()则会已影响原集合。
print(primeNums.union(oddNums))
/// 差集
/// subtracting() 不影响原集合，subtract() 影响原集合
print(oddNums.subtracting(primeNums))

// 用字典和高阶函数计算字符串中每个字符的出现概率
Dictionary("hello".map { ($0, 1) }, uniquingKeysWith: +)
```

这块有道非常经典的题目“2Sum”，题目大概是：给出一个整型数组和一个目标值，判断数组中是否有两个数之和等于目标值。这道题我是在 leetcode 上做的，刚开始根本就不知道用 Set ，折腾了很久，到后边慢慢就变好了。还有个变种题目，还要把当前的下标拿到，此时可以加入字典进行解题。


## 字符串和字符
Swift 的字符串类型于 Foundation 的 `NSString` 类型进行了无缝桥接，我们可以不用经过类型转换，可以直接在 `String` 中调用 `NSString` 的方法。

### 多行字符串字面量
如果我们需要跨越多行的字符串，可以使用多行字符串字面量：
```Swift
let quotation = """
The White Rabbit put on his spectacles.  "Where shall I begin,
please your Majesty?" he asked.

"Begin at the beginning," the King said gravely, "and go on
till you come to the end; then stop."
"""
```

如果文本太长，可以在适当的地方添加上反斜杠：
```Swift
let softWrappedQuotation = """
The White Rabbit put on his spectacles.  "Where shall I begin,\
please your Majesty?" he asked.

"Begin at the beginning," the King said gravely, "and go on\
till you come to the end; then stop."
"""
```


### 字符串（String）是值类型
Swift 编译器优化了字符串的使用，实际拷贝只会在需要的时候才进行。在 Swift 中字符串不同于其他语言（包括 OC），它是**值类型**而非引用类型。

```Swift·
// 检测字符串是否由数字构成
func isStrNum(str: String) -> Bool {
    return Int(str) != nil
}
```


### 可拓展的字形群集 & 字符串索引
在底层，Swift 中的原生 `String` 类型是由 `Unicode 标量` 构造而来的，而 `Unicode` 是一个在不同书写系统中编码，表示和处理文本的国际标准。它使我们能够以一种标准化形式表示几乎任何语言中的任何字符，Swift 中 `String` 和 `Character` 都完全符合 `Unicode` 标准的。

每一个 Swift 的 `Character` 类型代表一个 **可拓展** 的字符集，而拓展字符集由可以由多个不同的 `Unicode` 标量组成，这就意味着相同字符的不同表示需要占据不同的内存空间去存储，因此，在字符串的各种表示中 Swift 字符占据的内存不并不一样，这样造成的结果就是，字符串的字符数量并不能通过遍历该字符串去计算。

所以，`count` 属性返回的字符个数不会一直都与包含相同字符的 `NSString` 的 `length` 属性返回的字符个数相同，因为 `NSString` 的长度是基于 **UTF-16** 表示的字符串所占据的 16 位代码单元的个数决定，而不是字符串的字符集个数决定。

这也就不难理解，为什么 Swift 必须通过 `beginIndex` 或者 `endIndex` 等索引属性或方法来寻找字符在字符串中的位置，而不能像 `NSString` 那般通过确定的下标直接获取。

### inout
`inout` 关键字是按值传递，然后再写回原变量，而不是按引用传递。

### 给出一个字符串要求将其按照单词顺序进行反转
工程写法：

```Swift
let str = "the sky is blue"

var strArray = str.split(separator: " ")
strArray = strArray.reversed()

for s in strArray {
    print("\(s) ", terminator: "")
}
```

书中写法：
```Swift
fileprivate func _reverse<T>(_ chars: inout [T], _ start: Int, _ end: Int) {
    var start = start, end = end
    
    while start < end {
        _swap(&chars, start, end)
        start += 1
        end -= 1
    }
}

fileprivate func _swap<T>(_ chars: inout [T], _ p: Int, _ q: Int) {
    (chars[p], chars[q]) = (chars[q], chars[p])
}


func reverseWord(s: String?) -> String? {
    guard let s = s else {
        return nil
    }
    
    var chars = Array(Substring(s)), start = 0
    _reverse(&chars, 0, chars.count - 1)
    
    for i in 0 ..< chars.count {
        if i == chars.count - 1 || chars[i + 1] == " " {
            _reverse(&chars, start, i)
            start = i + 2
        }
    }
    
    return String(chars)
}

let str = "the sky is blue"
print(reverseWord(s: str) as! String)
```

## 链表
### 链表的基本概念
```Swift
// 链表节点
class ListNode {
    var val: Int
    var next: ListNode?
    
    init(_ val: Int) {
        self.val = val
        self.next = nil
    }
}

// 链表
class List {
    var head: ListNode?
    var tail: ListNode?
    
    // 头插法
    func appendToTail(_ val: Int) {
        if tail == nil {
            tail = ListNode(val)
            head = tail
        } else {
            tail!.next = ListNode(val)
            tail = tail!.next
        }
    }
    
    // 尾插法
    func appendToHead(_ val: Int) {
        if head == nil {
            head = ListNode(val)
            tail = head
        } else {
            let temp = ListNode(val)
            temp.next = head
            head = temp
        }
    }
}
```

有这么一道题目：给出一个链表和一个值 x ，要求将链表中所有小于 x 的值放到左边，所有大于或等于 x 的值放到右边，并且原链表的节点顺序不能变。例如 1 -> 5 -> 3 -> 2 -> 4 -> 2 ，给定 x=3 ，则要返回 1 -> 2 -> 2 -> 5 -> 3 -> 4 。

这道题的难点在于不能改变链表中原节点的位置，所以肯定不能只要一条链表，因为这样代码会写得非常难以理解（要保证原节点位置不变），所以根据书中给出的思路，我们可以先把问题简化，给定一个链表，只保留比给定值小的节点，这个方法写完后，我们再利用同样的思路在 else 分支中补上比给定值的代码即可。

```Swift
unc getLeftList(_ head: ListNode?, _ x: Int) -> ListNode? {
    // dummy 相当于是个哨兵，一直盯着头节点看！
    let dummy = ListNode(0)
    var pre = dummy, node = head
    
    while node != nil {
        if node!.val < x {
            pre.next = node
            pre = node!
        }
        node = node!.next
    }
    
    pre.next = nil
    return dummy.next
}

func partition(_ head: ListNode?, _ x: Int) -> ListNode? {
    let prevDummy = ListNode(0), postDummy = ListNode(0)
    var prev = prevDummy, post = postDummy
    
    var node = head
    
    // 尾插法处理左边和右边
    while node != nil {
        if node!.val < x {
            prev.next = node
            prev = node!
        } else {
            post.next = node
            post = node!
        }
        node = node!.next
    }
    
    // 防止构成环
    post.next = nil
    // 左右拼接
    prev.next = postDummy.next
    
    return prevDummy.next
}

let node0 = ListNode(1)
let node1 = ListNode(5)
node0.next = node1
let node2 = ListNode(3)
node1.next = node2
let node3 = ListNode(2)
node2.next = node3
let node4 = ListNode(4)
node3.next =  node4
let node5 = ListNode(2)
node4.next = node5

//var head = getLeftList(node1, 3)
var head = partition(node0, 3)

while head != nil {
    print(head!.val)
    head = head!.next
}
```

### ===
在 OC 中可以通过 `==` 来进行两个对象指针的判定，在 Swift 中提供的是另一个操作符 `===` ，用来判断两个 `AnyObject` 是否为同一个引用。

### 快行指针
快行指针是解决“链表成环”问题的较好思路，只需耗费 O(1) 空间，O(n) 时间即可。

```swift
func hasCycle(_ head: ListNode?) -> Bool {
    var slow = head
    var fast = head
    
    while fast != nil && fast!.next != nil {
        slow = slow!.next
        fast = fast!.next!.next
        
        if slow === fast {
            return true
        }
    }
    return false
}

let node0 = ListNode(1)
let node1 = ListNode(5)
node0.next = node1
let node2 = ListNode(3)
node1.next = node2
let node3 = ListNode(2)
node2.next = node3
let node4 = ListNode(4)
node3.next =  node4
let node5 = ListNode(2)
node4.next = node0

print(hasCycle(node0))
// true
```

在来看道题：删除链表中倒数第 n 个节点。例： 1 -> 2 -> 3 -> 4 -> 5 , n = 2，返回 1 -> 2 -> 3 -> 5。给定的 n 小于等于链表的长度。

```Swift
func removeNthFromEnd(head: ListNode?, _ n: Int) -> ListNode? {
    guard let head = head else {
        return nil
    }
    
    let dummy = ListNode(0)
    dummy.next = head
    var prev: ListNode? = dummy
    var post: ListNode? = dummy
    
    for _ in 0 ..< n {
        if post == nil {
            break
        }
        post = post!.next
    }
    
    while post != nil && post!.next != nil {
        prev = prev!.next
        post = post!.next
    }
    
    prev!.next = prev!.next!.next
    return dummy.next
}

var node0 = ListNode(1)
let node1 = ListNode(5)
node0.next = node1
let node2 = ListNode(3)
node1.next = node2
let node3 = ListNode(2)
node2.next = node3
let node4 = ListNode(4)
node3.next =  node4
let node5 = ListNode(2)
node4.next = node5

var head = removeNthFromEnd(head: node0, 3)
while head != nil {
    print(head!.val)
    head = head!.next
}
```


-----

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

在 C++ 中可以使用 **虚继承** 的方式解决，详情可看 ·[https://www.cnblogs.com/BeyondAnyTime/archive/2012/06/05/2537451.html](https://www.cnblogs.com/BeyondAnyTime/archive/2012/06/05/2537451.html)

* **菱形问题**：多个父类实现了同一方法，子类无法判断继承哪个父类的情况。`Java` 中可用 `interface` 的方式解决， `Swift` 中可用 `protocol` 的方式解决。

18. 从 xib 或 sb 拖拽出来的控件设置为 `weak` 是因为对应的 `view` 已经强引用它了，其生命周期和 `view` 是一致的了，除非 `view` 被释放，否则该控件不会被释放；而代码自定义控件，要设置为 `strong` 。