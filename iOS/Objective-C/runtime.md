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
