---
title: More-页面传值
date: 2018-01-30 21:41:25
tags:
- iOS
- 页面传值
---

这篇文章来梳理一番在iOS页面间的传值方式，说到页面传值，不管在任何平台开发中都是一个非常的重要的事情，这就让我想起了当初大一那会儿对Qt还不够熟悉，居然对两个Window之间的传值用了一个全局变量来实现，然后在其它Window中显式声明一个extern标记变量作为数据源。emmm，现在想起来当初真的是蠢洁又扇凉。

在iOS中的页面传值方式主要以下六种：
1. 属性传值（一般）
2. 单例传值（一般）
3. NSUserDefault传值（一般用作用户偏好设置）
4. 代理传值（常用）
5. block传值（常用）

在以上六种传值方式中，1、3、4使用得最为广泛，而且广泛存在与iOS SDK和各种第三方库中。

---

### 属性传值（正向传值）

属性传值不太常用于页面传值，因为通过属性传值有些时候需要顾及到各个页面的调用时机次序，反而通过属性设置一些页面的特殊功能标识的作用会更大。

举个🌰🍐！！！我们首先要进行跳转的第二个页面设置属性值propertyPassValue。

```ObjC
@interface NextViewController : UIViewController

@property (nonatomic, copy) NSString* propertyPassValue;

@end
```

重写propertyPassValue的set方法，属性传值的写法还有其它方式，如果我们对相关的对象做了lazy load，那么我们应该在lazy load block中进行赋值。
```ObjC
- (void)setPropertyPassValue:(NSString *)propertyPassValue {
    _propertyPassValue = propertyPassValue;
    self.tipsLabel.text = propertyPassValue;
}
```

此时我们通过实例化NextViewController，通过push或者present模态推出该vc即可看到对应的Label显示出propertyPassValue内容。

### 单例传值（可正可反）

单例传值，说是传值实际上应该说是利用了单例的“持久化”状态来达到持久化属性的一种方法，单例应该说是一种设计模式，单例是一把非常好使的“武器”，放在会使的人手里，单例会变成一把利剑，能够很好的解决跨多个页面的值传递问题，但是如果放在不熟悉的人手里，很有可能给他这个一高整个项目就散发着弄弄的java味了。🙂。（关于设计模式还会有一篇文章专门对其做了总结，大家可以持续关注。）

单例从字面上来理解，肯定是不同于往先alloc再init一个对象，最常见的用法就是NSUserDefault。（这个后边再说）

```ObjC
+ (instancetype)shareInstance {
    static instanceModel *model = nil;
    if (!model) {
        model = [[instanceModel alloc] init];
    }
    return model;
}
```
通过以上代码我们就创建出来了一个单例，实际上单例就是利用了static标识符告诉Xcode这个instanceModel变量要放在静态存储区，静态存储区是内存在程序编译的时候就已经分配好的，这块内存在App整个运行期间都将存在。它主要存放静态数据、全局数据和常量。

instanceModel是单拎出来的一个NSObject对象，以上代码只是创建出来一个存在整个App运行期间的单例，当我们把App从后台kill掉后，整个单例也就不存在了，因为此时App的运行期已经结束了，如果你要持久化数据那就得使用其它方法（比如存文件）。

```ObjC
    self.tipsLabel.text = [instanceModel shareInstance].instanceString;
```

instanceString是多加的一个NSString对象，我们通过其起到传值的作用。

### NSUserDefault传值（可正可反）

NSUserDefault是苹果原本用于提供用户偏好设置的，但是我大天朝神奇的程序员怎么可能听命与你？所以在实际开发中NSUserDefault有时候起到了鬼斧神工的作用，并且我们还可以使用它来作为页面之间传值的工具。

```ObjC
// 设置NSUserDefault对应的k-v
[[NSUserDefaults standardUserDefaults] setObject:@"NSUserDefaults传值" forKey:@"NSUserDefaults"];
// 数据同步
[[NSUserDefaults standardUserDefaults] synchronize];
```
而使用NSUserDefault也非常的简单，

```ObjC
self.tipsLabel.text = [[NSUserDefaults standardUserDefaults] objectForKey:@"NSUserDefaults"];
```

NSUserDefault底层是个文件，被苹果存在了对应的App沙盒中，只有我们去使用过它才会出现在沙盒中。（沙盒实际上就是对应的App文件夹）因此除非我们把这个App删掉和手动清洗掉对应的数据，否则我们对NSUserDefault设置的value将一直存在，同时它也是数据持久化的一直方式。在使用NSUserDefault作为页面间传值的工具时，千万要注意记得要手动显式调用NSUserDefault的数据同步方法，而且使用的时候尽量避免同时对一个key进行同时“写”操作。（不过也没人这么无聊对一个用户偏好key同时写吧？🙂）

### 代理传值（反向传值）

代理传值我可以拍着胸脯说这绝对是目前iOS中使用范围最广和使用次数最多的传值方式，而且基本上每一个VC的编写都会接触到代理，而且代理传值对于我自己来说也是一个非常常用的反向传值方式。

同时代理也是一种设计模式，第一次接触代理模式的同学（包括我）都会有些摸不着头脑，虽然说是和日常生活中找中介租/买房子非常像，但是转换到代码层面就是迷迷糊糊。🙂。先po一张图，

<img src="http://7xszq8.com1.z0.glb.clouddn.com/ww%20%285%29.png" width = "100%" height = "100%" align=center />

从上图中我们可以看到，需求方相当于是代理制定者，而受理方则是代理实现者（说得我都懵逼了😂）。直接瞅代码吧，

```ObjC
// 协议
@protocol NextViewControllerDelegate <NSObject>

- (void)passValueOfProtocol:(NSString *)string;

@end

@interface NextViewController : UIViewController

@property (nonatomic, weak) id<NextViewControllerDelegate> vcDelegate;

@end
```
从以上代码中，我们看到了制定代理使用@protocol关键字即可，@protocol中的内容可以认为是“合同内容”，我们还需要声明一个id指针变量vcDelegate，这个变量可以认为是“合同书”本身，而如何让需求方发布这个协议或者合同呢？

```ObjC
[_vcDelegate passValueOfProtocol:self.tipsLabel.text];

```

在需要的地方通过id指针变量_vcDelegate调用passValueOfProtocol方法即可，参数即为要填入合同书的内容🙂。通过显式调用以上方法即可完成所谓的“填写合同书”环节。接下来，我们要在对应的类中的遵守协议（合同书）即可使用需求方填入合同书中的内容。

```ObjC
// 写明要遵守的协议
@interface ViewController () < NextViewControllerDelegate>
@end
```

```ObjC
NextViewController *vc = [[NextViewController alloc] init];
// 写明要受理的代理对象
vc.vcDelegate = self;
```

```ObjC
// 要实现的代理方法
- (void)passValueOfProtocol:(NSString *)string {
    self.tableView.headTitleLabel.text = string;
}
```
受理方遵守协议的过程千万要注意设置代理对象，`vc.vcDelegate = self;`，如果你忘了设置对应的代理对象，而只是实现了协议方法，这就相当于我们把合同给了对方而已，对方并未**签名**，这份合同对甲乙双方实际上是无效的！因此千万别忘了设置代理对象。通过以上步骤，我们即可从需求方拿到数据，从而显示在代理方，是一种非常有效（但是有些麻烦）的反向传值方法。

### block传值

block，是苹果这几年来强烈推荐的回调传值方式。你可以认为是C中匿名函数，C++/java中的lambda表达式，JS中的闭包等。只不过在iOS中换了个说法称之为block（我是这么认为的🙂）。

```ObjC
@interface NextViewController : UIViewController

@property (nonatomic, copy) NSString* propertyPassValue;
@property (nonatomic, weak) id<NextViewControllerDelegate> vcDelegate;
@property (nonatomic, copy) void (^passValueBlock)(NSString *);

@end
```

在此我们把block作为了一个属性，而赋值block，
```ObjC
self.passValueBlock(self.tipsLabel.text);
```
我们只需要在对应的地方给block传入NSString类型参数即可。而使用block，我们在对应的VC中参考以下代码即可获取到对应的值。

```ObjC
 NextViewController *vc = [[NextViewController alloc] init];
  vc.passValueBlock = ^(NSString *string) {
    self.tableView.headTitleLabel.text = string;
};             
```
以上只是block的简单使用，关于block更加细节的一些用法，大家可以参考[这篇文章(操蛋的block语法🙂)](http://fuckingblocksyntax.com/)。

### 通知传值

通知也是一把利器，尤其是涉及跨多个页面间的传值时它的好处就提现出来了，而且最重要的是在项目中巧妙的使用通知能够打破以往纵向传递的繁杂性，从而达到一种“星型”发射状的模型。之所以这么说是因为通知可以说是最切合真正的面向对象的核心，真正的面向对象语言——smalltalk，如果没记错的话，smalltalk应该是一切面向对象语言的开山鼻祖吧😀。面向对象的真正核心应该是“一切操作皆消息”，也就是说，就算是简单加法、减法也是通过“发送消息”完成的，而不是像各种课本及老师说的什么继承、多态等等这些外壳，这些外壳用C也能够实现，根本就不算是面向对象语言的标志，反观现在比如C++、java这些当初跟smalltalk分道扬镳的语言，现如今的更新也都是在越来越像smalltalk罢了。（包括ruby🙂），OC就是基于smalltalk的封装，不过很多东西也没拿完。

而在iOS中实现通知得益于苹果爸爸的高度封装能力，把很多问题都给我们搞定了，

```ObjC
[[NSNotificationCenter defaultCenter] postNotificationName:@"notify" object:nil userInfo:@{@"notify" : self.tipsLabel.text}];
```

通过以上代码，我们就注册了一个名为notify的通知，其带有了一个key为notify且value为self.tipsLabel.text的字典。使用通知需要有一个通知中心作为沟通的桥梁，发送通知的object为告诉通知中心要把这个通知消息发送给哪个对象，如果想要群发消息通知，那就nil。

而接收通知，我们需要给当前类添加一个观察者，用于监听对应name的通知。
```ObjC
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(notifyMesg:) name:@"notify" object:nil];
```
以上代码中的object参数为接受哪个对象发送而来的消息，如果我们想接收任何对象发送而来的消息那就nil吧。接着，我们在处理消息通知的notifyMesg:方法中取得消息实体，
```ObjC
- (void)notifyMesg:(NSNotification *)notify {
    self.tableView.headTitleLabel.text = notify.userInfo[@"notify"];
}
```

当然，我们对当前类添加了观察者去监听某个通知，那就要在适当的实际去取消监听，当然你完全可以不用移除通知，但是如果多个消息通知重叠在一个项目中时，很容易就导致消息的监听迷之问题，因为一旦上升到通知层面的debug那就不是线性的了，可想而知难度得有多大。🙂。一般来说我们会在当前类的dealloc方法中移除通知。

