本篇文章为我在日常coding过程中使用OC进行了一些骚操作或者被虐得很惨的记录，可能会记得比较乱，因为有时候我也不知道应该怎么分类，但是我会努力哒！(๑•̀ㅂ•́)و✧

1. 实例对象所属的类成为类对象，而类对象所属的类被成为元类`MetaClass`。

  类的类型为`Class`类型，而`Class`类型为一个`objc_class`结构体类型指针。该结构体大致成员变量如下所示：
  ```c

  struct objc_class {
    Class isa;
    Class super_class;
    const char *name
    long version;
    long info;
    long instance_size;
    struct objc_ivar_list *ivars;
    struct objc_method_list *methodLists;
    struct objc_cache *cache;
    struct objc_protocol_list *protocols;
  }

  ```  
  * **isa** 指针是和Class同类型的objc_class结构体指针，类对象的指针指向其所属的类，即元类（上文已经说过啦~），元类中存储着类对象的类方法，当访问某个类的类方法时会通过该isa指针从元类中寻找方法对应的函数指针；
  * **super_class** 为该类所继承的父类对象，如果该类已经是最顶层的根类则为NULL（比如NSObjc或NSProxy）；
  * **ivars** 是一个指向objc_ivar_list类型的指针，用来存储每一个实例变量的地址；
  * **info** 为运行期使用的一些标识，比如`CLS_CLASS(0x1L)`该类为普通类，`CLS_META(0x2L)`表示该类为元类；
  * **methodLists** 为存放该类的方法列表，根据info中的标识信息，存储的方法可为实例方法或类方法；
  * **cache** 用于缓存最近使用的方法。系统在调用方法时先去cache中search，为空时才回重新去methodLists中寻找。

2. 实例对象是类对象`alloc`或`new`操作创建的，该操作会拷贝实例所属的类成员变量，但并不拷贝类定义的方法。
  * 调用实例方法时，系统会根据`isa`指针去类和父类的方法列表`methodLists`中寻找与发送的消息对应的`selector`指向的方法。
  * 任何带有以指针开始并指向类结构的结构都可以被看做`objc_object`。
  * 在OC中（或者一个面向对象语言）一个对象最重要的特点是可以给它发送消息。

  总结以上两点，我做了一张图说明，希望能帮助大家稍微的理清这其中的关系，如下：

  <img src="https://i.loli.net/2018/04/20/5ad9a2650b34d.jpeg" width = "100%" height = "100%" align=center />

  * 当发送消息给实例对象时，“消息”是在寻找这个对象的类的方法列表。当发送消息给类对象时，“消息”是在寻找类的元类方法列表。

  * 元类，其也是一个对象，我们同样也可以调用它的方法。所有的元类都使用根元类作为他们的类，那根元类的元类呢？对，就是根元类的元类就是它们自己，其`isa`指针指向它自己。此处再放一张图来帮助大家梳理一遍这其中的关系，如下所示：

    <img src="https://i.loli.net/2018/04/20/5ad9a48da9cdc.jpeg" width = "100%" height = "100%" align=center />

    * 实例对象的`isa`指针指向该实例对象的类，类的`isa`指针指向元类；
    * 类的`super class`指向其父类，上文已经说了，如果该类为根类，则其父类为nil；
    * 元类的`isa`指针指向根元类（注意不是父元类），若该类本身就是根元类，则指向其本身。
    * 元类的`super class`指向父元类，若该类为根元类则指向该根类。

3. `id`类型。其为`objc_object`结构类型的指针，该类型的对象可转换为任何一种对象，类似C语言中的`void *`。

4. `@property` = `ivar` + `gettter` + `setter`，即属性是添加了 **存取方法** 方法的成员变量。

5. 针对`String`的`copy`和`strong`的理解。
  * 若为可变数据类型，即当前为`NSMutableString`，分别设定`copyString`和`strongString`，进行的赋值操作如下所示：
  ```objc
    @property (copy) copyString;
    @property (strong) strongString;

    NSMutableString *string = @"2333";
    copyString = string;
    strongString = string;

    [ts appendString:@"4666"];

    NSLog(@"%@", copyString); // 2333
    NSLog(@"%@", strongString); // 23334666
  ```
  由上可见，当用`strong`修饰可变数据类型`NSMutableString`时，其会因为原始数据的值的改变而改变。

 * 当为不可变数据类型，即`NSString`时，分别设定`copyString`和`strongString`，进行如上所示操作时两者均不会改变，来看`copyString`的setter方法实现：

     ```ObjC
     - (void)setCopyString:(NSString *)copyString {
       [_copyString release];
       // 拷贝了参数内容，创建了一块新的内存
       _copyString = [copyString copy];
     }
     ```

     接下来看`strongString`的setter方法实现：
   ```ObjC
     - (void)setStrongString:(NSString *)strongString {
       [_strongString release];
       // copy了指针
       [strongString retain];
       _strongString = strongString;
     }
 ```

6. `#import`和`#include`的区别
  * `#import`确保引用的文件只会被引用一次，不会引起交叉编译；
  * 两者均把后边的文件名所代表的文件拷贝到指令所在的文件；
  * `#import`会链入该头文件的全部信息，包括实例变量和方法；
  * `@class`只告诉编译器其后跟内容为类的名称，不用管该类是如何定义的，且一般在头文件中使用。
  * 使用`#import`的优点：可解决头文件中的循环依赖问题

7. `nonatomic`和`atomic`
  * `atomic`：默认值。只有一个线程可以访问，至少在当前线程的读取是安全的，但由于使用 **同步锁** 开销过大，会损耗性能（macOS性能较好，可不考虑该问题），其是一个原语操作，编译器会自动生成一些互斥加锁的相关代码，避免变量读写不同步等的问题。保证必须当前一个线程执行完相关的`setter`方法后，另一个线程才执行`setter`；
  * `nonatomic`：不保证`setter/getter`的原语执行，故可能会取到不完整的值。因此我们可以得到一个约束：**多线程环境下的原子操作是非常非常非常必要的！！！**
  * 解释两个概念：原语和原语操作，
    - **原语**：内核或微内核提供核外调用的过程或函数，是一段用机器指令编写的、完成特定功能的程序代码，且执行过程不允许中断；
    - **原子操作**：在多进程（或线程）的操作系统中不能被其它进程（或线程）打断的操作。当该操作不能完成时，必须回到该操作之前的状态，原子操作是不可拆分的，原子操作是中断且安全的。其本质实现还用到了 **自旋锁** ，自旋锁大概的意思是，当被其它对象使用时，待使用对象一直在循环等待并查看是否被释放。

8. `@Property`关键词及其相关关键字的理解：
  * 根据被修改的可能性，、@Property中关键字的排列推荐为：原子性、读写性、内存管理特性；
  * **原子性：** automatic和nonautomatic。决定了该属性是否为原子性的，即在多线程的操作中，不能被其它线程打断的特性，一旦使用了该变量的操作不能被完整执行时，将会回到该变量操作之前的状态，但原子性即automatic因为是原语操作（保证setter/getter的原语执行），会损耗性能，在iOS开发中一般不用，而在macOS开发中随意。
  * **读写性：** readOnly和readWrite。默认为readWrite，编译器会帮助生成serter/getter方法，而readOnly只会帮助生成getter方法。 // 此处可拓展，非要修改readOnly修饰的变量怎么办，可用KVC，又可继续拓展KVC相关知识。
  * **内存管理特性：** assign、weak、strong、unsafe_unretained。
    - assign：一般用于值类型，比如int、BOOL等（还可用于修饰OC对象）；
    - weak：用于修饰引用类型（弱引用，只能修饰OC对象）；
    - strong：用于修饰引用类型（强引用）；
    - unsafe_unretained：只用于修饰引用类型（弱引用），与weak的区别在于，被unsafe_unretained修饰的对象被销毁后，其指针并不会被自动置空，此时指向了一个野地址。

9. `Block`的理解：
  * Block与函数指针非常类似，但Block能够访问函数以外、词法作用域以外的外部变量的值；
  * Block不仅实现了函数的功能，还携带了函数的执行环境；
  * Block实际上是指向结构体的指针；（可参考[这篇文章](https://www.cnblogs.com/yoon/p/4953618.html)）
  * Block会把进入其内部的基本数据类型变量当做常量处理。】
  * Block执行的是一个回调，并不知道其中的对象合适被释放，所以为了防止在使用对象之前就被释放掉了，会自动给其内部所使用的对象进行retain一次。
  * Block使用`copy`修饰符进行修饰，且不能使用`retain`进行修饰，因为`retain`只是进行了一次回调，但block的内存还是放在了栈空间中，在栈上的变量随时会被系统回收，且Block在创建的时候内存默认就已经分配在栈空间中，其本身的作用域限于其创建时，一旦在超出其创建时的作用域之外使用，则会导致程序的崩溃，故使用`copy`修饰，使其拷贝到堆空间中，block有时还会用到一些本地变量，只有将其copy到堆空间中，才能使用这些变量。

10. 循环引用的几种情况：
  * **NSTimer**：
  * **block**：
  * **delegate**：

11. Objective-C中的反射机制
  Foundation框提供了反射API，可通过API把字符串转为`SEL`操作，且因为OC的动态性，这些操作都发生在**运行时**。我们可以在运行时选择需要创建的实例，并动态的选择调用方法，这些操作可以由服务器下发的参数进行控制。

  什么意思呢？比如说，我们可以通过后台推送过来的数据进行动态跳转，跳转到页面后再根据返回的数据执行对应的操作。比如，
  ```json
  // 假设返回了这些数据
  {
      "vc" : "homeViewController",
      "methon" : "reloadData",
      "propertys" : [
        {"url" : "www.baidu.com"},
        {"title" : "百度"},
      ],
  }
  ```
  我们可以这么写，
  ```ObjC
    Class class = NSClassFromString(dict[@"vc"]);
    UIViewController *vc = [[class alloc] init];
    NSDictionary *parameter = dict[@"propertys"];
    [parameter enumerateKeysAndObjectsUsingBlock:^(id  _Nonnull key, id  _Nonnull obj, BOOL * _Nonnull stop) {
        if ([vc respondsToSelector:NSSelectorFromString(key)]) {
            [vc setValue:obj forKey:key];
        }
    }];
    [self.navigationController pushViewController:vc animated:YES];
    SEL selector = NSSelectorFromString(dict[@"method"]);
    [vc performSelector:selector];

  ```

12. iOS 中有以下几种随机数取法：
  * `random()` ：伪随机数，需要自己加种子，否则每次产生的随机数都是一样的。，一般都是以当前时间做种子。不推荐；
  * `arc4random()` ：真随机数，但如果要获取一定范围内的随机数，需要自己做模运算；
  * `arc4random_uniform()` ：真随机数，只需要填入范围上边界数字即可获取到对应范围内的真随机数。

持续更新中.....

13. 如果在 `category` 中声明了一个和原类同名的方法，或和该类的另一个 `category` 中的方法同名，那么在运行时究竟哪个方法会被执行是不确定的。

14. 用 `Objective-C` 编写的程序不能直接编译成可令机器读懂的机器语言，而是在程序运行时，通过运行时（runtime）把程序转译成可令机器读懂的机器语言；用 `C++` 编写的程序，编译时就直接编译成了可令机器读懂的机器语言。这也就是为什么把 `Objective-C` 视为一门动态开发语言的原因。

15. 开发语言的三个不同层次：
* 传统的面向过程的语言开发。
* 改进的开发面向对象的语言。
* 动态开发语言。

16. **在头文件中尽量减少其他头文件的引用**

* 通过 `#import` 修饰符来建立被引用类的指针。用 `#import` 建立类之间的复合关系时，也暴露类所引用类的实体变量和方法，可以使用 `@class` 来告诉编译器：这是一个类。
* `#import` 引用类同一个头文件，或者这些文件是依次引用的，如 A->B、B->C、C->D，当最开始的那个头文件有变化时，后面所有引用它的类都需要重新编译，如果自己的类有很多的话，这将耗费大量时间，使用 `@class` 则不会。
* 注意「类循环引用」的问题。

16. 尽量使用模块方式与多类建立复合关系
* `#include` 做的事情其实就是简单的复制、粘贴，将目标 `.h` 文件中的内容一字不落的复制到当前文件中。
* `#import` 实际上与 `#include` 是一样的，不过 `Objctive-C` 为了避免重复引用可能带来的编译错误（比如 B 和 C 都引用类 A，D 又同时引用了 B 和 C，这样 A 中定义的东西就在 D 中被定义了两次，造成重复）而加入了 `#import`，保证每个头文件只会被引用一次。`#import` 的实现是通过对 `#ifdef` 一个标志进行判断，然后再引入 `#define` 这个标志来避免重复引用。
* **使用 `.pch` 预编译头文件**不实用。确实能缩短编译时间，但在工程中不能随处访问的东西，却都暴露了。
* 使用模块（`Modules`）来解决。在 `Build Settings` 中搜索 `Modules` 修改为 `YES`。默认情况下，模块功能在所有的新工程中都是开启的，语法修改如下：
    ```objc
    @import UIKit;
    @import MaKit;
    ```
* 如果只导入框架中自己需要的部分可以这么做：
  ```objc
  @import UIKit.UIView;
  ```
* 在技术上，我们不需要把所有 `#import` 都换成 `@import`，因为编译器会隐式的转换它们，但建议尽可能的使用新语法。

17. 尽量避免使用 `#define`，`#define` 预处理指令不包含任何的类型信息，仅仅是在编译前做替换操作，它们在重复定义时不会发出警告，容易在整个程序中产生不一样的值。

18. 处理隐藏的返回类型，优先选择 `instancetype` 而非 `id`。

19. `NSLog` 并不是向 Xcode 控制台中输出信息，而是向苹果系统日志（Apple System Log，ASL）中输出错误信息。因此我们要把 `NSLog` 看作是 `printf` 和 `syslog` 的结合体：在调试时将消息发送到 Xcode 控制台，在设备上运行时将消息发送到系统全局日志。然后 `NSLog` 记录的数据就可以被任何拿到物理设备的人获取。在发布应用之前把 `NSLog` 从代码中删除。
- 在发布版本中禁用 `NSLog`：
  ```objc
  #ifdef DEBUG
  #define NSLog(...) NSLog(__VA_ARGS__);
  #else
  #define NSLog(...)
  #endif
  ```
20. `%x` 和 `%n` 分类符对攻击者来说非常有用。

21. `strcpy` 缓冲区溢出攻击。如果输入超出了固定字符长度的字符，超出的那部分字符就会覆盖相邻栈变量的内存，这就意味着，攻击者可以重载函数的返回地址，让程序执行恶意代码，攻击者可以把恶意代码直接放在输入当中，也可以放在内存中的其它位置。

22. 防止 `XSS` 攻击：
  - 设置黑名单。告知用户哪些字符不允许输入。
  - 设置白名单。只允许输入哪些字符。
  - 显示字符时，选转为 `HTML` 字符串，再输出，保证了 `<` 和 `>` 等特殊字符被转译。

23. 空指针（NULL 指针）是指没有存储任何内存地址的指针。野指针，是指向“垃圾内存”的指针。

24. `@autoreleasepool`。在 ARC 下，没有办法手动通知系统对某个对象执行 `autorelease`，当给一个对象设置了 `__autorelease` 修饰符修饰时，相当于这个对象在 MRC 下给这个对象发送了 `autorelease` 消息，注册到了 `autorelease pool` 中。

25. 指针地址对齐。为了加快内存的 CPU 访问，包括 macOS 和 iOS 在内的几乎所有系统架构都使用了**指针地址对齐**概念，其指在分配堆中的内存时往往采用**偶数倍**或以 2 为倍数的内存地址作为地址边界。

26. **标记指针**。由于指针地址对齐和 64 位超大地址的出现，指针地址仅仅作为内存的地址比较浪费，故可以在指针地址中保存或附加更多的信息，进而引入了**标记指针**的概念。其指的是那些指针中包含特殊属性或信息的指针。

27. 利用标记指针处理 `NSNumber`，直接可以把实际的值保存到指针中，而无须再去访问堆中的数据，提高内存访问速度和整体运算速度。

28. 标记指针堆 `isa` 指针的优化。在 OC 中所有的类都继承自 `NSObject`，因此每个对象都有一个 `isa` 指针指向它所属的类。在 32 位和 64 位环境下， `isa` 指针会产生不同的变化。
  - 在 32 位环境下，对象的引用计数都保存在一个外部的表中，对引用计数的增减操作都要先锁定这个表，操作完成后才解锁，效率比较慢。
  - 在 64 位环境下，`isa` 指针也是 64 位，实际作为指针的部分只用到其中的 33 位，剩余的部分会运用到**标记**指针的概念。其中的 19 位将保存对象的引用计数，这样对引用计数的操作只需要**原子**的修改这个指针即可。如果引用计数超过 19 位，才会将引用计数保存到外部表，情况较少，故效率可以大大提高。

29. 兼容 32 位和 64 位环境下代码编写事项（其实没啥用了）。
  - **不要将长整型数据赋予整型**。
  - **善用 `NSInteger` 来处理 32 位和 64 位之间的转换**。`NSInteget` 在 32 位运行时是 32 位整数，在 64 位运行时是 64 位整数。
  - **创建数据结构要注意固定大小和对齐**。
  - **选择一种紧凑的数据表示类型**。

30. 常量字符串和一般字符串的区别。
  - 由于编译器的优化，相同内容的常量字符串的地址值是完全相同的。
  - 如果使用常量字符串来初始化一个字符串，那么这个字符串也将是相同的常量。
  - 对常量字符串永远不要 `release`。

31. 在访问集合时要优先考虑使用快速美剧。
  - 使用快速枚举，枚举更安全。因为枚举会监控枚举对象的变化，如果在枚举的过程中枚举对象发送变化会抛出一个异常。
  - 多个枚举可以同时进行，因为在循环中被循环对象是禁止修改的。

32. 同一数组（`NSArray`）可以保存不同的对象，但不能存储 `float`、`int`、`double` 等基本类型和 `nil`，否则存储基本类型都会被设置为 0，不能存储 `nil` 是因为数组必须用 `nil`。

33. `autorelease pool` 提供一种机制：让对象延迟 `release`。这个对象放弃所有权，但又想避免立即释放（如何函数的返回值）。有些时候，可能会使用自己的 `autorelease` 池块。
  - 通常情况下，应该使用 `release`，而不是 `autorelease`，只有在不适合立即回收对象的情况下，才应该使用 `autorelease`。
  - 当返回一个新创建的（拥有）的对象时，应该使用 `autorelease` 而不是 `release` 来释放所有权。
  - 对于拥有 `alloc` 返回的对象而言，失去释放所有权之前，应先失去对该对象的引用。

34. 对象的 `isa` 实例变量指向对象的类。

35. `alloc` 和 `init` 不仅进行对象的内存分配，还要对它的 `isa` 实例变量和 `retain count` 初始化。

36. 对象销毁或者被移除一定考虑所有权的释放。
  - 从集合中移除对象，集合要释放对被移除对象的所有权。
  - 防止出现父对象被释放前而子对象的所有权已经释放。
  - 释放对象前，要确保其他对象对该对象的所有权已经释放。

37. 编译指令。

    指令 | 含义
    ---- | ----
    `@private` | 变量只限于声明它的类访问
    `@protected` | 变量可以被声明它的类及继承该类的类使用。没有明确指定访问范围的变量默认为 `@protected`
    `@public` | 变量可以在任何位置访问
    `@package` | 变量可以在同一个 `framework` 中访问

38. 动态属性。在 OC 2.0 中增加了一个新的关键字 `@dynamic`，用于定义动态属性。动态属性相对于 `@synthesis` 不是由编译器自动生成 `setter` 和 `getter`，也不是由开发者自己写的 `setter` 或 `getter`，而是在运行时动态添加的 `setter` 和 `getter`。

    实现动态属性需要在代码中覆盖 `resolveInstanceMethond` 来动态添加 `name` 的 `setter` 和 `getter`。这个方法在每次找不到方法时都会被调用。`NSObject` 的默认实现就是抛出异常。

39. 在覆盖基类的方法决定是否调用 `super`，基于打算如何重新重写方法，可以注意以下亮点：
  - 如果打算**补充**基类实现的行为，调用 `super`。
  - 如果打算**替换**基类实现的行为，不调用 `super`。

40. 在 OC 中，所有的方法都是虚方法。实现纯虚方法依赖协议来实现。

41. 类的对象支持归档和解档，该类必须遵循 `NSCoding` 协议；必须实现对对象进行编码（`encodeWithCoder:`）和解码（`initWithCoder：`）的方法。

42. `KVC` 的实现原理主要是运用了 `isa-swizzling` 技术（类型混合指针机制），通过其来实现内部查找定位。`isa` 指针指向的是对象的类，这个类也是一个对象，有自己的权限，根据类的定义编译而来。类对象负责维护一个方法调度表，该表本质上是由指向类方法的指针组成的，类对象中还保留一个基类指针，该指针也有自己的方法调度表和基类，还有所有通过继承得到的公共和保护的实例变量。`isa` 指针对消息分发机制和 Cocoa 对象的动态能力很关键。

43. 在 `swift` 中可以使用 `extension` 对类的实现进行拆分，在 `ObjC` 中可以选择使用 `category` 对类的实现进行拆分。

44. 内省是对象揭示自己作为一个运行时对象的详细信息的一种能力，这些详细信息包括对象在继承树上的位置、对象是否遵循特定的协议，以及是否可以响应特定的消息。

45. `isEqual` 方法先检查指针的等同性，然后是类的等同性，最好调用对象的比较器进行比较。

46. 使用 `new` 创建对象时，实际发生了两个步骤：第一个步骤，为对象分配内存，也就是说对象活动存储其实例变了的内存快；第二步，自动调用 `init` 方法，初始化对象使其处于可用状态。没有被初始化的指针都是 `nil`。

47. 使用类拓展隐藏私有信息。

48. 父对象应该强引用子对象，子对象变量应该弱引用父对象。

49. 对一些不支持 `__weak` 引用的类，可通过 `Unsafe Unretained` 引用来暗度陈仓。

50. 类别的一些内容：
  - 子类体现了类的上下级关系，而类别是类间的平级关系。
  - 类别具有替换特性，如果类别方法与类内某个方法具有同样的方法签名，类别里的方法将会替换类的原有方法。
  - 类别是为类**增加外部方法**的话，类扩展是用做类的**内部拓展**。

51. 类簇。基于抽象工程模式，可以用于隐藏实现的详细细节，为调用者提供一个简单的接口。看《编写高质量代码：改善 OC 程序的 61 个建议》第 48 个建议。

52. `alloc` 方法使用应用程序默认的虚存区，区是一个按页对齐的内存区域，用于存放应用程序分配的对象和数据。除了分配内存之外，还做 了：
  - 将对象的保持数设置为 1。
  - 使初始化对象的 `isa` 实例变量指向对象的类。对象类是一个根据类定义编译得到的运行时对象。
  - 将其他所有的实例变量初始化为 0（或与 0 等价的类型，比如 `nil`，`NULL`，`0.0`）

53. 在创建对象时，通常应该在处理之前检查返回值是否为 nil。

54. 需要 OC 对象的存取器来帮助进行引用计数。

55. 通过调用 `[xxx setValueForKey:xxx]` 要比 `[xxx setValue:xxx]` 要慢得多，因为编译器无法检查传递给 `valueForKey:` 的字符串是否有效，同时效率也变成了原来的 5%，如果需要获取值的运行参数，则使用 `[xxx performSelector: xxx]` 是直接消息发送速度的 2 倍，比 `valueForKey:` 快 10 倍。

56. KVO 和 Cocoa 绑定是基于 KVC 的，其速度不会很快。

57. **OpenUDID 是什么？**实际上是跟着 app 走，每次重装 app 都会重新生成一个 id，一般都会把它放到 keychain 中进行系统级的持久化。

58. `NSUserDefaults` 实际上是在 Library 文件夹下生成一个 plist 文件，如果该文件太大，读取时会比较耗时，因为加载的时候是直接全部 load 到内存中。头条主端通过测试，200 多个缓存数据，通过符号断点 `+[NSUserDefaults standardUserDefaults]` 确定最早一次的  `+load()` 从执行到结束耗时 1.8ms

59. `mach_absolute_time` 获取当前时间的「纳秒」，需要 mach 库。

60. 忽略警告的大概做法:
	```swift
	#pragma clang diagnostic push
	#pragma clang diagnostic ignored "-Wunguarded-availability"
	[UNUserNotificationCenter currentNotificationCenter].delegate = [TTNotificationCenterDelegate sharedNotificationCenterDelegate];
	#pragma clang diagnostic pop
	```
  
61. 想要成为一个 `AppDelegate` 需要：
	* 继承 `UIResponder` 和 `UIApplicationDelegate` 协议
	* 在 `main.m` 中通过 `UIApplicationMain` 进行初始化

62. ARC 在「编译」的时候插入内存管理代码

63. bitcode。上传 app store 的时候实际上上传的是一个「平台无关的代码」，用户在下载 app 的时候 App Store 会根据用户的机型翻译成对应的机器代码进行下发。

64. Link 链接
* 解决依赖
* 确定地址引用
* Mach-O 结构
* 生成可执行文件

65. Xcode 提示一个符号找不到声明是在「语法解析生成 AST」时出错。

66. 打包提示`missing symbols`是在「链接」出错。

67. OC 中的 ARC 是在编译的**机器码**生成支持的。

68. 代码中使用了静态库中的某个方法，是在**链接**时确定符号地址的

69. OC 中在方法里跑另外一个 方法/代码块的做法：
	```objc
    - (void)_enterFullScreenWithAngle:(CGFloat)angle animted:(BOOL)animated {
      void (^animation)() = ^{
        
      };
      
      animation()
    }
    ```

70. NSCache
  * 线程安全，键不会发生复制操作
  * 拥有 LRU，不需要自己写缓存置换算法，如果用 NSCache 去做的话，就需要了
  * `NSCache` 可以设置缓存中的对象数量


71. 为什么在 iOS 上用 nonamatic，macos 不用？
  * iOS同步锁开销很大，会带来性能问题。一般情况下不要求必须是原子性的，因为使用了也并不能确保真正的线程安全。如果一个线程多次读取某属性值的过程中有别的线程在同时改写该值，那么即便使用了atomic，也还是会读到不同的属性值。

72. category 和 extension
  * OC 的 category 相当于 Swift 的 extension
  * OC 的 extension 加私有方法，直接在创建各种 UIView 的时候就已经带上了

73. __auto_type
  * 自动类型推倒
  * https://pspdfkit.com/blog/2017/even-swiftier-objective-c/
  ```objc
  #define let __auto_type const
  #define var __auto_type 

  let anElegantView = [UIView new];
  let something = (TheType *)array.firstObject;
  var something = array.firstObject;
  ```

74. Enum 关联对象
  ```swift
  enum CSSColor {
      case named(ColorName)
      case rgb(UInt8, UInt8, UInt8)
  }

  var color1 = CSSColor.named(.black)
  var color2 = CSSColor.rgb(0xAA, 0xAA, 0xAA)
  switch color2 {
  case  let  .named(color):
        print("\(color)")
  case .rgb(let r, let g, let b):
        print("\(r), \(g), \(b)")
  }
  ```

75. 集合的可变类，属性不使用copy修饰符的原因？
  * [文章解释](https://juejin.im/post/5bedfdaa6fb9a049cd53c56d)
  * 在 ARC 下，编译器在合成 `setter` 方法时，走的是 `copy`，就会把原先的例如 `NSMutableArray` 变成了 `NSArray`，再执行 `addObject` 方法时会找不到方法而报错。
  * copy 默认调用的是 `copyWithZone `
  * [相关 session](https://developer.apple.com/videos/play/wwdc2017/411/)

  76. 预编译阶段处理的宏定义，在组件进行二进制化后会失效，特别是某些依赖 `DEBUG` 宏的调试工具，在二进制化之后就不可见了。针对这种情况可以单独抽出一个类来替换宏，把需要用到宏的地方归类到一个中间者去完成，并且不让这个中间者去做二进制化。

    ```objc
    // TDFMacro.h
  @interface TDFMacro : NSObject
  + (BOOL)enterprise;
  + (BOOL)debug;

  + (void)debugExecute:(void(^)(void))debugExecute elseExecute:(void(^)(void))elseExecute;
  + (void)enterpriseExecute:(void(^)(void))enterpriseExecute elseExecute:(void(^)(void))elseExecute;
  @end

  // TDFMacro.m
  @implementation TDFMacro
  + (BOOL)enterprise {
  #if ENTERPRISE
      return YES;
  #else
      return NO;
  #endif
  }

  + (BOOL)debug {
  #if DEBUG
      return YES;
  #else
      return NO;
  #endif
  }

  + (void)debugExecute:(void (^)(void))debugExecute elseExecute:(void (^)(void))elseExecute {
      if ([self debug]) {
          !debugExecute ?: debugExecute();
      } else {
          !elseExecute ?: elseExecute();
      }
  }

  + (void)enterpriseExecute:(void (^)(void))enterpriseExecute elseExecute:(void (^)(void))elseExecute {
      if ([self enterprise]) {
          !enterpriseExecute ?: enterpriseExecute();
      } else {
          !elseExecute ?: elseExecute();
      }
  }
  @end
  ```

77. 使用 `printf` 语句输出内容可以保值某段计算代码不会被视为死代码，然后被计算机优化掉。

78. 选取相片后，通过 `asset` 拿到具体的 `UIImage`，可以通过以下字符串拼接 `URL` 获取：
```objc
if (item.asset) {
    NSString *assetID = [item.asset.localIdentifier substringToIndex:(item.asset.localIdentifier.length - 7)];
    imageURL = [NSURL URLWithString:[NSString stringWithFormat:@"assets-library://asset/asset.jpg?id=%@&ext=jpg", assetID]];
} else {
    imageURL = [NSURL URLWithString:item.fullpathLink];
}
```

79. OC 自定义 `setter` 和 `getter` 命名 
```objc
// You can customize the getter and setter names instead of using default 'set' name:
@property (getter=lengthGet, setter=lengthSet:) int length;
```

80. `valueForKeyPath` 为什么慢，因为走的是 hash

81. 一个 Button 的点击事件 @selector 如何优雅的传递多参数
* 使用 block 捕获，包装一下

82. OC 没有办法将方法标为私有。其每个对象都可以响应任意消息，而且可在运行期检视某个对象所能直接响应的消息，跟进给定的消息查出其对应的方法，这一工作要在运行期才能完成。
	* 定义私有方法时最好在方法前加上前缀 `p_xxx`

83. 在使用协议的时候，每次都要在原类中检查 delete 是否实现了该协议中的某个方法，可以选择使用标志位的方法去缓存检查的值。
* 这么做的前提是检查的方法会被调用很多次，并且也确实是因为检查了很多次的这些方法造成了性能瓶颈，我们采取优化它。

84. 分类中的方法是直接添加到类里面的，他们就好比这个类中的固有方法。
* 将分类方法加入类中的这一操作是在运行期系统加在分类时完成的，在运行期系统会把分类中所实现的每个方法都加入到类的方法列表中。
* 如果类中已有此方法，分类中又实现了一遍，那么分类中的方法将会覆盖类中实现的相关方法，而另外一个分类中的方法由覆盖了这个分类的方法。
* 多次覆盖的结果，以最后一个分类为准。

85. OC 对象所占内存在 release 后，只是放回“可用内存池”，如果执行 `NSLog` 时尚未覆写对象，那么该对象仍然有效。

86. 避免悬挂指针，在释放完对象后置 `nil`

87. 遇到保留环时，在「垃圾回收器」环境下会把三个对象全都收走，在引用计数架构中，需要使用「弱引用」来打破。

88. CoreFoundation 对象不归 ARC 管理。

89. C++ 对象由于抛出异常会缩短其生命周期，所以发生异常时必须析构，不然就会泄漏。

90. 自动释放池用于存放那些需要在稍后某个时刻释放的对象。清空自动释放池时，系统会向其中的对象发送 `release` 消息。

91. GCD 机制中的线程默认都有自动释放池，每次执行“事件循环”时就会将其清空，故不需要在 GCD 部分创建自动释放池。

92. 向已回收的对象发送消息是不安全的，但这么做有时可以，有时不行，完全取决于对象所占内存有没有为其他内容所覆写。
* 在没有崩溃的情况下，那块内存可能只复用了其中一部分，该对象中的某些二进制数据依然有效。
* 还有一种可能，那块内存恰好为另外一个有效且存活的对象所占据，运行期系统会把消息发送到新对象哪里，新对象可能会应答，也可能不会，如果不能应答就崩溃。

93. 单例对象的「保留计数」都很大很大。

94. 浮点数的 `NSNumber` 对象保留计数是 1。
`Block` 会把它所捕获的所有变量都拷贝一份，拷贝的不是对象本身，而是指向这些对象的指针变量。

95. crash 可以分为 四类
* OC Exception
* Mach Exception
* Unix Signal
* C++ Exception

![crash 类型](https://i.loli.net/2019/08/02/5d442268a8d4d81154.png)

96. 发生 OOM 时app 在前台的话，会 crash

97. 使用 `GCD` 执行异步派发时，需要拷贝块。

98. 想让几段代码按顺序执行，或者执行 A 代码块时，B 代码块不能执行，可以考虑用 GCD 的串行队列，能够保证一个代码块在执行时，另外一个代码块在等待执行。

99. **读可以并行读，写要求顺序写。**在队列中，`barrier` 队列（栅栏队列）必须单独执行，不能与其他块并行。这只对并发队列有意义，因为串行队列中的块总是按顺序执行。并发队列如果发现接下来要处理的块是个 `barrier` 块（栅栏块），那么就一直要等待当前所有并发块都执行完毕，才会单独执行这个栅栏块。栅栏块执行完毕后，才按照正常的方式继续向下处理。

100. `performSelector` 系列方法在内存管理方法容易有遗漏。它无法确定将要执行的选择子具体是什么，因而 ARC 编译器也就无法插入适当的内存管理方法。

101. `sizeThatFits` 与 `sizeToFit`
*  `sizeToFit` 可以自动计算宽高，并且还会修改视图的 `frame`
* `sizeThatFits` 只能自动计算宽高

102. `NSDateFormatter` 会造成性能损耗
* 过度的创建其用于 `NSDate` 和 `NSString` 的转化，会造成性能下降。
* 如果需要 `NSDateFormatter` 进行频繁的操作，推荐对其缓存起来。

103. 为什么 `cornerRadius` 会造成性能下降
* 其会触发离屏渲染
* 指图像在绘制到屏幕前，需要先进行一次渲染，之后才绘制到当前屏幕
  * `alloc` 一块内存，进行渲染。
  * onScreen 和 offScreen 之间上下文切换代价比较大。

104. 转屏逻辑可以写在 `layoutSubView时` 中。因为每次 `frame` 切换都会导致该方法的调用。
* 当时如果视图的 `frame` 为0，则不会被调用。

105. 每一个 `NSThread` 对象都是一个完整的线程。

106. 遍历集合的几种方式：
* `for`
* `NSEnumerator`
* `for-in`
* `块枚举法`

107. 实现缓存功能时优先选用 `NSCache` 而不是 `NSDictionary` 对象。因为其可以提供优雅的**自动删减**功能，且是**线程安全**的，与字典不同，不会拷贝键。还可以给其设置上线，用于限制缓存中的对象总个数及总成本。

108. 在**加载阶段**，如果类实现了 `load` 方法，那么系统就会调用它。分类里也可以定义此方法，类的 `load` 方法要比分类中的先调用。与其他方法不同，`load` 方法不参与覆写机制，也就是说，父类和子类都写了 `load` 方法，不会向上执行，各执行各的。

109. 首次使用某个类之前，系统会向其发送 `initialize` 消息。由于此方法遵从普通的覆写机制，所以通常应该在里面判断当前要初始化的是哪个类，也就是说，如果父类写了该方法，子类没写，父类在执行该方法时，子类也会被执行。
* 无法在编译器设定的全局常量，可以放在 `initialize` 方法里初始化

110. GCD 相当于是个线程池。

111. 队列设置优先级时，低优先级队列任务可能会阻塞高优先级，尽量用默认优先级，因为可能会发生「优先级反转」。
  > 优先级翻转是当一个高优先级任务通过信号量机制访问共享资源时，该信号量已被一低优先级任务占有，因此造成高优先级任务被许多具有较低优先级任务阻塞，实时性难以得到保证。

112. 异步操作同步返回可以使用 GCD 的 `dispatch_semaphore`。

113. 子线程发通知主线程收不到，因为在哪个线程发送通知就在哪个线程接收通知，有两种做法可以解决。
  * 在发起通知的时候检查一遍当前线程是不是主线程，不是主线程切回主线程发送。
  * 在收到通知的地方统一归并到主线程中。

114. 如何判断当前线程是否为主线程？每个线程都会有自己的名字，没有名字的线程则会打印出 `(null)`，因此可以参考 `SDWebImage` 的做法：

```objc
#ifndef dispatch_main_async_safe
#define dispatch_main_async_safe(block)\
    if (strcmp(dispatch_queue_get_label(DISPATCH_CURRENT_QUEUE_LABEL), dispatch_queue_get_label(dispatch_get_main_queue())) == 0) {\
        block();\
    } else {\
        dispatch_async(dispatch_get_main_queue(), block);\
    }
#endif
```

其中运用到了 `strcmp` 这个 C 语言函数，其对两个字符串判断的逻辑如下：

* 字符串1=字符串2，返回0。
* 字符串1>字符串2，返回一个正整数。
* 字符串1<字符串2，返回一个负整数。

115. 转屏逻辑可以写在 `layoutSubView时` 中。因为每次 `frame` 切换都会导致该方法的调用。
	* 当时如果视图的 `frame` 为 0，则不会被调用

116. 使用以下名称开头的方法名意味着自己生成的对象只有自己持有：
* `alloc`
* `new`
* `copy`
* `mutableCopy`

当对方法进行命名时，如果出现了下列类似的方法名，也意味着自己生成并持有对象：
* `allocMyObject`
* `newThisObject`
* `copyThis`
* `mutableCopyYourObject`

117. 调用类似 `[NSMutableArray array]` 方法使得取的对象存在，但自己不持有对象：

  ```objc
  - (id)objc {
    id obj = [[NSObject alloc] init];
    // 自己持有对象
    [obj autorelease];
    // 取得的对象存在，但自己不持有对象
    return obj;
  }
  ```
118. 想要让自己原本并不持有的对象，变为持有，给该对象加上一个 `[obj retain]` 进行持有。

119. 重复释放或释放了自己不持有的对象，会导致崩溃。

120. `dealloc` 方法到底什么时候调用？
每次执行单次 `[obj release]` 或系统自动执行统一 `release` 时，判断某个对象的 `retainCount` 是否为 0，为 0 则手动调用 `[self release]` 方法，`free()` 掉该对象的内存。

121. Apple 通过散列表来管理引用计数。表 `key` 为内存块地址的散列值，`value` 为内存块的引用计数。这么做可以通过计数表的各个记录追溯到各对象的内存块，即使出现故障导致对象占用的内存块损坏，但只要引用计数没有被破坏，就能够确认各内存块的位置。

122. OC 中同时重写 `setter` 和 `getter` 需要把属性改为 `@dynamic` 修饰，告知 Xcode 不要帮我自动生成。

123. 该方法返回 `autorelease` 对象。

  ```objc
  id array = [NSMutableArray arrayWithCapacity:1];
  // 相当于
  id array = [[NSMutableArray alloc] initWithCapacity:1] autorelease];
  ```

  124. 调用 `[obj autorelease]` 本质上是调用：

  ```objc
  - (id)autorelease {
    [NSAutoreleasePool addObject:self];
  }
  ```

  为了能够高效地运行应用程序中频繁调用的 `autorelease` 方法，使用了 `IMP Caching` 的机制，在框架初始化的时候对其结果值进行缓存。运行效率一般是其它方法的 2 倍。

124. 如果嵌套生成多个 `NSAutoreleasePool` 对象，`[obj autorelease]` 会使用最内侧的 `NSAutoreleasePool` 对象。

125. `NSAutoreleasePool` 的 `drain` 方法实现细节：
* 调用 `drain` 方法，本质上是在调用 `[self dealloc]` 方法。
* 调用 `[self dealloc]` 方法，本质上是在调用 `[self emptyPool]` 和 `[array release]`，清空 pool 和自己本身管理对象数组的 release。
* 调用 `[self emptyPool]` 本质上是在循环遍历对象数组中 `autorelease` 添加进来的对象的 `[obj release]` 方法。

总的来说，就是会让每个对象都会被 `release`。

126. 对 `[NSAutoreleasePoolObjc autorelease]` 会怎样？

运行时会发生异常，因为 `NSAutoreleasePool` 类中已经重载了 `autorelease` 方法，运行时会报错。

127. `id` 和对象类型在没有明确指定所有权修饰符时，默认为 `__strong` 修饰符。

128. `+load()` 在这个文件被装载时调用。只要是在 Compile Sources 中出现的文件总是会被装载，这与该类是否被用到无关，因此 `load` 方法总是在 `main()`  函数被调用。子类实现 `load` 方法时，会先调用父类的 `load` 方法。当类的加载是耗时或者需要消耗比较多的内存的时候,尽量不要在 `load` 方法里面做这些耗时的工作,因为这样会**增加App的启动时间**，降低用户的体验。由于调用load方法时的环境很不安全，我们应该尽量减少 `load` 方法的逻辑。另一个原因是load方法是线程安全的，它内部使用了锁，所以我们应该避免线程阻塞在 `load` 方法中。一个常见的使用场景是在 `load` 方法中实现 `Method Swizzle`

* 这个方法会在类的第一个方法调用前被调用。首先会先调用父类的 `initialize` 方法，如果子类没有实现 `initialize` 方法,那么父类会多次触发这个方法,为了避免这种情况的发生，可以在实现的方法里面添加一个判断。`initialize` 其实可以被认为是延迟加载的方法，类加载的时候并不会执行这个方法，只有当类实例化的时候，或者类的第一个方法被调用的时候才会执行这个方法。
* `load` 方法通常用来进行 `Method Swizzle`，`initialize` 方法一般用于初始化全局变量或静态变量。

129. `__autoreleasing` 一个有趣的地方：
```objc
NSError *error = nil;
BOOL result = [pbj performOperationWithError:&error];
```

该方法的声明为：

```objc
- (BOOL) performOperationWithError:(NSError **)error;
```

`id` 的指针或对象的指针会默认加上 `__autoreleasing` 修饰符，所以等同于以下代码：

```objc
- (BOOL) performOpertaionWithError:(NSError * __autoreleasing *)error;
```

但是如果是这样直接赋值的话，会产生编译器错误：

```objc
NSError *error = nil;
NSError **pError = &error;
```

因为赋值给对象指针时，所有权修饰符必须一致，修改：

```objc
NSError *error = nil;
NSError * __strong *pError = &error;
```

对于其它所有权修饰符也是一样的。

130. 以 `init` 开始的方法的规则要比 `alloc\new\copy\mutableCopy` 更严格。该方法必须是实例方法，并且必须要返回对象。返回的对象应为 `id` 类型或该方法声明类的对象类型，或者是该类的超类或子类。该返回的对象不注册到 `autoreleasepool` 上，基本只对 `alloc` 方法返回值的对象进行初始化处理并返回该对象。

131. 被 `__unsafe_unretained` 修饰的变量不属于编译器的内存管理对象。如果管理时不注意赋值对象的所有者，便有可能遭遇内存泄漏或程序崩溃。

132. `id` 和 `void *` 类型对象的转换可以基于 `__bridge` 进行，但 `__bridge` 转换不改变对象的持有状况。

133. `id *` 类型默认为 `id __autoreleasing` 类型。

134. 使用 `__weak` 修饰符的变量所引用的对象被废弃，将自动赋值为 `nil`，且该对象会被自动注册到 `autoreleasepoll`中。

135. `__weak` 有关的这行代码

```objc
id __weak obj1 = obj0
```

实现细节最后会调用一个 `objc_storeWeak(&obj1, 0);` 函数，该函数把第二参数的赋值对象的地址作为 `key`，将第一参数的附有 `__weak` 修饰符的变量的地址注册到 `weak` 表中。如果第二个参数为 0，则把变量的地址从 `weak` 表中删除。

`weak` 表和引用技术表相同，作为**散列表**被实现。

136. 释放 `__weak` 修饰的对象时，最后会调用 `objc_clear_deallocating` 函数，其动作为：

* 从 `weak` 表中获取废弃对象的地址为 key 的记录；
* 将包含在记录中的所有附有 `__weak` 修饰符变量的地址，赋值为 `nil`；
* 从 `weak` 表中删除该记录；
* 从引用计数表中删除废弃对象的地址为 key 的记录。

可见，大量使用 `__weak` 修饰符的对象会消耗对应的 CPU 资源，只需要在避免循环引用时使用该修饰符。

137. 每使用一个被 `__weak` 修饰符修饰的对象时，都会被加入到 `autoreleasepool` 中，为了避免这个问题，可以先把其用 `__strong` 修饰符修饰的对象接一下。

138. 做埋点时如果不能保证取的值都是存在的话，使用字典 `setValue：forKey：` 中 value 能够为 nil，但是当 value 为 nil 的时候，会自动调用`removeObject：forKey` 方法。


139. 现在的 `blocks` 并没有实现对 C 语言数组的截获，可以使用指针解决。

```objc
const char *text = "hello";
void (^blk)(void) = ^ {
  printf("%c\n", text[2]);
}
```

140. `block` 会被转换为 C 语言源码编译。同时也为 OC 的对象。

141. 所谓“截获自动变量值”意味着在执行 `block` 语法时，`block` 语法表达式所使用的自动变量值被保存到 `block` 的结构体实例中（`block` 本身）。

142. iOS 13 中返回的 `device token` 变化异常。
iOS 13 之前，基本上都是这么去获取的 device token：

```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSString *deviceTokenString = [[[[deviceToken description]
                                     stringByReplacingOccurrencesOfString: @"<" withString: @""]
                                    stringByReplacingOccurrencesOfString: @">" withString: @""]
                                   stringByReplacingOccurrencesOfString: @" " withString: @""];
}
```

```shell
<baac4207 4eb30c26 264b43b7 7cedf1ba 643da57b 1bdd6356 9ef43b5c aa5b6c30>
```

iOS 13 后，变为了

```objc
{length = 32, bytes = 0x778a7995 29f32fb6 74ba8167 b6bddb4e ... b4d6b95f 65ac4587 }
```


所以我们需要这么做，并且该代码也是向下兼容的：

```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(nonnull NSData *)deviceToken
{
    const unsigned *tokenBytes = [deviceToken bytes];
    NSString *deviceTokenString = [NSString stringWithFormat:@"%08x%08x%08x%08x%08x%08x%08x%08x",
                                   ntohl(tokenBytes[0]), ntohl(tokenBytes[1]), ntohl(tokenBytes[2]),
                                   ntohl(tokenBytes[3]), ntohl(tokenBytes[4]), ntohl(tokenBytes[5]),
                                   ntohl(tokenBytes[6]), ntohl(tokenBytes[7])];

}
```

143. URL 中的 `?` 是保留字符，所以在判断 URL 中最后一位是不是 `?` 没有必要，直接看当前 URL 是否包含 `?`，即可判断。

144. 通过 `NSStringFromSelector` 来获取方法选择器名字。

145. 使用 `Asset Catalog` 管理资源图片，其中添加的 2x 图和 3x 图会在提交 app store 时被创建成不同的变体以减小 App 安装包的大小，用户下载 app 时会拉取到不同的变体文件文件。

146. 可以使用 AppCode 的 `inspect Code` 选项来初步检查出无用的类和方法，但注意会有一些问题。

147. 本地的 @2x 和 @3x 转成 webp 以后，调用的时候是否要判断设备分辨率，根据不同的设备分辨率调用不同倍数的 webp。

148. 在多个 Block 中使用 `__block` 变量时，因为最先会将所有的 Block 配置在栈上，所以 `__block` 变量也会配置在栈上。在任何一个 Block 从栈复制到堆时，`__block` 变量也会一并从栈复制到堆并被该 Block 所持有。当剩下的 Block 从栈复制到堆时，被复制的 Block 持有 `__block` 变量，并增加 `__block` 变量的引用计数。

149. 什么时候栈上的 Block 会被复制到堆上呢？
* 调用 Block 的 `copy` 方法；
* 将 Block 作为函数返回值；
* 将 Block 赋值给附有 __strong 修饰符 `id` 类型的类或 Block 类型成员变量时；
* 方法名中含有 `usingBlock` 的 Cocoa 框架方法或 GCD 的 API 中传递 Block 时。

150. 推荐调用 Block 的 `copy` 方法
* 将 Block 作为函数返回值；
* 将 Block 赋值给附有 __strong 修饰符 `id` 类型的类或 Block 类型成员变量时；
* 方法名中含有 `usingBlock` 的 Cocoa 框架方法或 GCD 的 API 中传递 Block 时。

151. 当监控系统内存的县城发现某 App 内存有压力，发出通知，内存有压力的 App 就会去执行对于的代理，也就是 `didReceiveMemoryWarning` 代理。通过这个代理，可以获得最后一个编写逻辑代码释放内存的机会。这段代码的执行，有可能会避免 App 被系统强杀。

152. iOS 系统内核里有一个数组，用于维护线程的优先级。这个优先级规定就是：内核用线程的优先级是最高的，操作系统的优先级其次，App 的优先级排在最后。且前台 App 的优先级高于后台 App。线程使用优先级时，CPU 占用多的线程的优先级会被降低。

153. 系统因为内存占用原因强杀 App 前，至少有 6s 的时间可以用来做优先级判断，`JetSamEvent` 日志在这段时间内生成。

* 在收到内存警告时，如何获取当前 app 的内存？

```objc
struct mach_task_basic_info info;
mach_msg_type_number_t size = sizeof(info);
kern_return_t kl = task_info(mach_task_self(), MACH_TASK_BASIC_INFO, (task_info_t)&info, &size);
```

```objc
float used_mem = info.resident_size;
NSLog(@" 使用了 %f MB 内存 ", used_mem / 1024.0f / 1024.0f)
```

154. NSLog 实际上是一个 C 函数。它的作用是输出信息到标准的 Error 控制台和系统日志中，在内部实现上，实际上是用 ASL （Apple System Logger）的 API，将日志消息直接存储在磁盘上。

155. ARC 有效时，`id` 类型以及对象类型变量必定附加所有权修饰符，却省为 `__strong`。

156. 推荐通过 `copy` 方法来持有 block，把 block 从栈上复制到堆上。 

157. **ARC 无效时**，`__block` 说明符被用来避免 Block 中的循环引用。因为 Block 从栈复制到堆时，不会被 `retain`，反之会被 `retain`。


158. `NSURLConnection`  发起请求后，所在的线程需要一直存活，以等待接收 `NSURLConnectionDelegate` 回调方法，但是网络返回的时间不确定，所以这个线程需要一直常驻内存中。

159. 线程保活：
* 使用 `NSRunLoop`
    * `runUntilDate:`
    * `runMode:beforeDate`

160. 在进行数据读写操作时，需要一段时间来等待磁盘响应，如果此时通过 GCD 发起一个任务，GCD 就会本着最大化利用 CPU 原则，会在等待磁盘响应这个空档，再创建一个新线程来保证能够充分利用 CPU。

161. 类似数据库这种需要频繁读写磁盘操作的任务，尽量使用串行队列来管理，避免因为多线程并发而出现内存问题。

162. 创建线程引发内存问题：
* 创建线程的过程需要用到物理内存，CPU 也会消耗时间
* 创建一个线程，系统需要为这个进程空间分配一定的内存作为线程堆栈。堆栈大小时 4KB，在 iOS 中主线程堆栈大小时 1MB，新创建的子线程堆栈大小是 512KB。
* 线程创建多了，CPU 在切换线程上下文时，还会更新寄存器，更新寄存器的时候需要寻址，而寻址的过程还会有较大的 CPU 消耗。

163. 除了加锁还有什么其它方法能够保证数据线程安全？
* 串行队列

164. 判断耗电量可以从 CPU 使用量入手。

165. `dispatch_after` 函数并不是在指定时间后执行处理，而只是在指定时间追加处理到 `Dispatch Queue`。

166. 通过 `Dispatch Group` 可以统一管理 GCD，在其中各个 GCD 执行完后处理或者设置等待时间。

167. 如果想提高文件读取速度，可以尝试使用 `Dispatch I/O`。