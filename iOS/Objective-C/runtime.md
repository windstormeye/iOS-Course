# Runtime
我终于来啃这个大骨头了，从开始学 iOS 开发起，`runtime` 和 `runloop` 一直挥之不去，网络上的各种资料写的关于这两个方向的内容也十分庞大，弄得我一直不敢碰（梳理）。趁着最近时间较充裕，学习学习 `runtime` 中的相关知识并加入到实际项目中，需要注意的是**本篇文章部分内容来自网络**。

## runtime 是什么？
`runtime` 是 `Objective-C`（后文简称 OC ） 的主要一个特性，从学习并使用 OC 脑海中就一直在回响着“它是一门动态语言，动态语言”，而在 OC 中实现动态特性所采用的技术或者实现方案就是 `runtime` ，它基本上是 C 和汇编语言构成，目前有 Apple 和 GNU 分别维护的两套 `runtime` 版本，但这两个版本大体上无差别，都在尽力的保持着一致。在 OC 2.0 中采用的是最新版本的 `runtime` 。

动态语言，相较于我们所认为的静态语言（注意：这不是说的强弱类型语言）一言以蔽之：在程序运行的过程中可以改变其结构，例如新的函数可以被引进，已有的函数可以被删除，可以给类添加新的属性等在结构上的变化。总是想办法把一些决定工作从编译链接推迟到运行时，也就是说只有编译器是不行的，还需要一个运行时系统（ runtime system ）来执行编译后的代码，这就是 `runtime` 存在的意义，是整个 OC 运行框架的一个基石。

但是 `runtime` 也有缺点，混乱的运行时代码会改变运行在其架构之上的所有代码。

## runtime 可以用来做什么？
在 OC 中，我们可以利用 runtime 做以下但不限于此的事情：
* 利用关联对象为分类增加伪属性；
* 利用 `Methon Swizzling` 交换方法；
* 利用 class_copyIvarList 实现 NSCoding 的自动归档解析；
* 利用 objc_allocateClassPair 、objcct_setClass 等 API 来实现 KVO Block；
* 利用消息转发机制实现多播委托（蹦床模式）；
* 字典模型之间的转换；
* 页面无侵入埋点；
* 监听 App 网络流量。

## 来玩玩 runtime 吧！
### 利用关联对象为分类增加伪属性
在项目的开发中，很多时候会遇到要为已经存在的类添加属性或方法，比如给 `NSString` 添加一个取当前时间戳的方法、给 `UINavigationBar` 添加自定义属性等。因为分类结构的特殊性，当我们需要做到给现有类添加属性时，分类并不会自动的为我们创建实例变量和存储方法，需要我们自己去实现存取方法：

通常推荐的做法是添加的属性最好为 `static char` 类型，当然最好是指针类型，通常来说该属性应该是常量、唯一的。

首先创建一个分类文件（下文不再赘述）：

缺图：
1 2 3 

在 `NSString+test.h` 文件中写下我们想要添加的属性：
```Objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface NSString (test)

@property (nonatomic, strong) NSString *title;

@end

NS_ASSUME_NONNULL_END
```

在 `NSString+test.m` 文件中写下对应的实现：
```Objc
#import "NSString+test.h"
// 注意引入 runtime.h
#import <objc/runtime.h>

@implementation NSString (test)

- (NSString *)title {
    return objc_getAssociatedObject(self, _cmd);
}

- (void)setTitle:(NSString *)title {
    // 注意要根据添加的属性不同选择不同的修饰符，此时我添加的属性是 NSString 类型，故为：OBJC_ASSOCIATION_COPY_NONATOMIC
    objc_setAssociatedObject(self, @selector(title), title, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

@end
```

这样就完成了给已有类添加额外属性的工作，接下来在我们需要用到的地方这么使用；

```Swift
#import "ViewController.h"
// 注意导入对应分类文件
#import "NSString+test.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSString *string = [NSString new];
    string.title = @"2333";
    
    NSLog(@"%@", string.title);
}

@end
```

## 为什么使用关联对象？
在代码中有些地方不适合拓展多个属性进行设置，同时也不适合写太多的冗余代码，从设计模式上看，需要改造现有的类。而想要在不增加类属性的方法去做到一些事情，可以通过 `category` 加分类的方法，但是 `category` 中做不到直接声明一个新的 `@property` 属性，因为在 `category` 中并没有为我们生成实例变量以及存取方法，需要我们手动实现。

所以一般使用关联对象为已经存在的类添加「属性」。在分类中，因为**类的实例变量的布局**已经固定，使用 `@property` 已经无法向固定的布局中添加新的实例变量（这样做可能会覆盖子类的实例变量），所以我们要用关联对象以及两个方法来模拟构成属性的三个要素

### 思考和总结：
其实这应该从实际角度出发，还是得回到最开始遇到的问题上。我们需要的是对现有类在不增加额外的多余代码前提下，增加新的属性（或者其它的一些做法），那对于一个属性来说，其核心就是三点：

* 生成实例变量 `_property`
* 生成 `getter` 方法 `- property`
* 生成 `setter` 方法 `- setProperty`

所以，想办法通过 `runtime` 去完成就完事了。

如果我们把属性看作是「通过方法访问的实例变量」，那很明显在分类中因为类的实例变量布局已经固定，所以是不能添加新的属性的。但如果我们把属性看做是一个「存取方法以及存取值的容器的集合」，那这是 OK 的。因为分类中对属性的实习其实只是实现了一个看起来像属性的接口而已。实现细节可以看[这篇文章](https://draveness.me/ao)

### 使用 Runtime 可以完成的一些事情
在运行期间动态创建一个类 MyRuntimeClass，里面含有一个方法print，打印出 Hello World! [详情可见](http://southpeak.github.io/2014/10/25/objective-c-runtime-1/)

### 给某个类动态添加上某个方法
```objc
    // 动态添加eat方法

    // 第一个参数：给哪个类添加方法
    // 第二个参数：添加方法的方法编号
    // 第三个参数：添加方法的函数实现（函数地址）
    // 第四个参数：函数的类型，(返回值+参数类型) v:void @:对象->self :表示SEL->_cmd
    class_addMethod(self, @selector(eat), eat, "v@:");
```