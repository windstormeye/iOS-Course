---
title: 招一个靠谱的iOS实习生（附参考答案）
date: 2018-04-08 20:11:03
tags:
- iOS
---

以下是我列出来的能够帮助大家招到一个 **靠谱的iOS实习生** 需要掌握的点，再次说明下情况：

1. 此份题适用于电面和face to face，更加偏向于电面；
2. 能够较为流畅的说到每道题的点上，基本上可以认为是掌握了；
3. 考虑到电面过程中，对被电面者心理素质考验非常大，所以，我本人抵制电面过程中考算法（这是一个流氓行为）此套题不涉及任何关于算法方面知识。若有此需求，推荐找专门的在线OJ进行测评。


## 概念部分

### struct和class的区别？
* **struct中不能定义函数**（针对面向过程语言，例如C）// 可不用
* **使用大括号进行初始化** class和struct如果定义了构造函数，就不能用大括号初始化，若没有，则struct可以，class只有在所有成员变量均为public时才行。 // 可不用
* **默认访问权限** class默认成员访问权限为private；struct默认访问你权限为public。 // 重点
* **继承方式** class默认private；struct默认public。 // 重点

### 说出以下指针的含义：
```C
int **a : 指向一个指针的指针，该指针指向一个整数。
int *a[10] : 指向一个有10个指针的数组，每个指针指向一个整数。
int (*a)[10] : 指向一个有10个整数数组的指针。
int (*a)(int) : 指向一个函数的指针，该函数有一个整数参数，并返回一个整数。
```

### int 和 NSInterger 的区别：
* 以C语言举例，int和long的字节数和当前操作系统中的指针所占位数是相等的，也就是说，long的长度永远 ≥ int，并且我们需要去考虑此时是使用int还是long比较合适，会不会因为一时疏忽选择了int而导致位数不够造成溢出。
* 而在OC中使用 NSInterger ，苹果对其进行了一个宏定义的判断（cmd+鼠标左键进去看吧），这个宏定义会自动判断当前App运行的硬件环境，到底是32位机还是64位机等等，从而自动返回最大的类型，而不用我们去思考此时到底应该是用int还是long。

### 深拷贝和浅拷贝的区别：
* 浅拷贝：又称“指针拷贝”。不增加新内存，只增加一个指针指向原来的内存区域。
* 深拷贝：又称“内容拷贝”。同时拷贝指正和指针所指向的内存，新增指针指向新增内存。
* // 可对OC中的可变对象和不可变对象做拓展，此问题只是单纯的概念。

### 内存中的区域是怎么划分的？
* 用之前做的一张图进行描述：
  <img src="https://i.loli.net/2018/04/08/5aca1d572b81b.jpeg" width = "100%" height = "100%" align=center />

## 语言部分

### nil、Nil、NULL和NSNULL的区别：
* **nil：** 把对象置空，置空后是一个空对象且完全从内存中释放；
* **Nil：** 用nil的地方均可用Nil替换，Nil表示置空一个类；
* **NULL：** 表示把一个指针置空。（空指针）
* **NSNULL：** 把一个OC对象置空，但想保留其容器（大小）。

### category和extension的区别：
* **category：**为已知类增加新方法。
  - 新增方法被子类集成；
  - 新增的方法比原有类具备更高的优先级，且不可重名，防止被覆盖；
  - 不能增加成员变量。
* **extension：** 为当前类增加私有变量和私有方法，添加的方法是必须实现的。

### @Property关键词及其相关关键字的理解：
* 根据被修改的可能性，、@Property中关键字的排列推荐为：原子性、读写性、内存管理特性；
* **原子性：** automatic和nonautomatic。决定了该属性是否为原子性的，即在多线程的操作中，不能被其它线程打断的特性，一旦使用了该变量的操作不能被完整执行时，将会回到该变量操作之前的状态，但原子性即automatic因为是原语操作（保证setter/getter的原语执行），会损耗性能，在iOS开发中一般不用，而在macOS开发中随意。
* **读写性：** readOnly和readWrite。默认为readWrite，编译器会帮助生成serter/getter方法，而readOnly只会帮助生成getter方法。 // 此处可拓展，非要修改readOnly修饰的变量怎么办，可用KVC，又可继续拓展KVC相关知识。
* **内存管理特性：** assign、weak、strong、unsafe_unretained。
  - assign：一般用于值类型，比如int、BOOL等（还可用于修饰OC对象）；
  - weak：用于修饰引用类型（弱引用，只能修饰OC对象）；
  - strong：用于修饰引用类型（强引用）；
  - unsafe_unretained：只用于修饰引用类型（弱引用），与weak的区别在于，被unsafe_unretained修饰的对象被销毁后，其指针并不会被自动置空，此时指向了一个野地址。


### OC中如何定义一个枚举？
* 在OC中定义一个枚举有三种做法：
  - 因为OC是兼容C的，所以可以使用C语言风格的enum进行定义。
  - 使用`NS_ENUM`宏进行定义；
  - 使用`NS_OPTIONS`宏进行定义；

* `NS_ENUM`为定义通用性枚举，只能单选，`NS_OPTIONS`为定义位移枚举，可多选。 // 枚举为啥要这么分？因为涉及到是否使用C++模式进行编译有关。


### Block和函数的关系（对Block的理解）？
* Block与函数指针非常类似，但Block能够访问函数以外、词法作用域以外的外部变量的值；
* Block不仅实现了函数的功能，还携带了函数的执行环境；
* Block实际上是指向结构体的指针；（可参考[这篇文章](https://www.cnblogs.com/yoon/p/4953618.html)）
* Block会把进入其内部的基本数据类型变量当做常量处理。】
* Block执行的是一个回调，并不知道其中的对象合适被释放，所以为了防止在使用对象之前就被释放掉了，会自动给其内部所使用的对象进行retain一次。
* Block使用`copy`修饰符进行修饰，且不能使用`retain`进行修饰，因为`retain`只是进行了一次回调，但block的内存还是放在了栈空间中，在栈上的变量随时会被系统回收，且Block在创建的时候内存默认就已经分配在栈空间中，其本身的作用域限于其创建时，一旦在超出其创建时的作用域之外使用，则会导致程序的崩溃，故使用`copy`修饰，使其拷贝到堆空间中，block有时还会用到一些本地变量，只有将其copy到堆空间中，才能使用这些变量。


### deletegate需要weak修饰的原因？
* 以图说明，图中所表示的是VC对tableView的持有，如果此时的tableView.deletegate对VC也是强引用，会导致循环引用，同时也给了我们敲了警钟，当出现两个对象都是强引用时，万分小心！
<img src="https://i.loli.net/2018/04/08/5aca2289b3f16.jpeg" width = "100%" height = "100%" align=center />

### 解释一下这段代码的输出：
* 简写：
```objc
Computer : NSObject
Mac : Computer

@implementation Mac

NSLog(@"%@", [self class]);
NSLog(@"%@", [super class])

@end
```
* 二者都会输出Mac。
  - [self class]：当使用self调用方法时，从当前类方法列表中找，若没有则再去父类中找。调用[self class]时，会转化成`objc_msgSend`函数，其定义为`id objc_msgSend(id self, SEL op, ...)`，第一个参数是Mac实例，但其并无`-(Class)class`方法，此时去父类Computer中寻找，发现也没有，再去其父类`NSObject`中找，找到了！返回的就是`self`其本身，可猜测其方法实现如下：
  ```objc
  - (Class)class {
    return object_getClass(self);
  }
  ```

  - [super class]：从父类方法列表中开始找，调用父类方法。当调用[super class]时，转换成`objc_msgSendSuper`函数，其定义为`id objc_msgSendSuper(struct objc_super *super, SEL op, ...)`，而`struct objc_super`结构体的定义为：
    ```objc
    struct objc_super {
      __unsafe_unretained id receiver;
      __unsafe_unretained Class super_class;
    }
    ```
    所以转换成`objc_msgSendSuper`函数后，第一步要先去构造`objc_super`结构体，结构的第一个成员`receiver`就是`self`，第二个成员是`(id)class_getSuperclass(object_getClass("Mac"))`，该函数输出的结果为super_class值，即`Computer`，第二步，则去`Computer`类中去找`- (Class)class`，发现并未找到，接着去NSObject中找，找到了！最后是使用了`objc_msgSend(objc_super->receiver, @selector(class))`去调用了，这个时候已经跟之前的[self class]调用输出结果重复了，返回结果还是`Mac`。


## iOS部分

### UITableView性能调优的方法：
* Cell重用：
  - 数据源方法优化：创建一个静态变量重用ID，例如：`static NSString *cellID = @"cellID";`防止因为调用次数过多，static保证只创建一次，提高性能（感觉性能的提升可以忽略不记emmm）
  - 缓存池获取可重用Cell的两个方法：`dequeueReusableCellWithIdentifier:(NSString *)ID`会查询可重用Cell，若注册了原型Cell则能够查询到，否则为nil，故需要先判断`if(cell == nil)`
  - `dequeueReusableCellWithIdentifier:(NSString *)ID indexPath:(NSIndexPath *)indexPath`，使用之前必须通过SB/class进行可重用Cell的注册（registerNib/registerClass），不需要判断nil，一定会返回cell，若缓冲区Cell不存在，会使用原型Cell重新实例化一个新Cell。
* 尽量使用一种类型的Cell：能够减少代码量，减小Nib文件的数量；保证只有一种类型的Cell，实际上App运后只有N个Cell，但若有M种Cell，则实际上运行最多却可能会是MxN 个。
* 善用hidden隐藏subview：把所有不同类型的view都定义好，通过cell的枚举类型变量及hidden显示/隐藏不同类型的内容，因为在实际快速滑动中，单纯的显示/隐藏subview比实时创建快得多。
* 提前计算并缓存Cell的高度。如果我们不预估行高，则优先调用`heightForRowAtIndexPath`获取每个Cell即将显示的高度，实际上就是要确定总的tableView.contenSize，最后才又接着调用`cellForRowAtIndexPath`，可以建一个frame模型，保存下提前计算好的cell高度。
* 异步绘制：这是目前最火的tableView性能调优方法，新浪微博是这么做的，可以使用`ASDK`这个库进行。
* tableView滑动时，按需加载：识别tableView静止或减速滑动结束后，异步加载，在快速滑动过程中，只按需加载目标方位内的Cell。
* 避免大量使用图片缩放、颜色渐变、透明图层、CALayer特效（阴影）等操作，尽量显示大小刚好合适的图片资源。

### 内存优化方案：
* 首先ARC。但要注意防止循环引用，避免内存泄露；
* 懒加载。延迟创建对象，用时再创建；
* 复用。比如tableView、collectionView单元格的复用；
* 巧妙使用单例，而不是全都使用单例！


### 单例的写法？
```objc
static User *user;
+ (User *)shareInstance {
  if (user == nil) {
    @synchronized(self) {
      // 加锁
      user = [User alloc] init];
    }
  }
  return user;
}


+ (User *)shareInstance {
  static dispatch_onec_t onecToken;
  dispatch_onece(&onceToken, ^{
    user = [User alloc] init;
  })
  return user;
}
```

### iOS的远程推送过程？
* 以图讲解：
<img src="https://i.loli.net/2018/04/08/5aca38905387f.jpeg" width = "100%" height = "100%" align=center />


### iOS中多线程的概念：（单问概念）
* 多线程优点：提高程序执行效率。缺点：开启线程需要一定的内存控件。
* 同步和异步：决定了要不要开启新的线程。同步：在当前线程中执行任务，不具备开启新线程能力；异步：在新线程中执行任务，具备开启新线程的能力。
* 并行和串行：决定了任务的执行方式。并行：多个任务并发（同时）执行，类似迅雷多任务同时下载；串行：一个任务执行完毕后，再执行下一个任务，类似一个一个下载。
* **重点：** 必须要明确iOS中只有一个主线程——UI线程，且不可将耗时任务放在主线程执行，否则会造成卡顿。




总结：再次说明，实际电面过程中，本套题只可作为参考，而且在电面过程中会有追问和等待的过程，所以最佳电面时间为三十分钟内最佳，嫌短？记好了！这招的是实习生，而且这还是电面！！！别学啥大公司的戾气，一个实习生的电面你要搞一个小时甚至快两个小时，没有必要。如果是face to face随你问一两个小时，但重点是这是电面！不要把这种“隔空喊话”的面试方式作为考核一个人是否具备对应的工作能力的最终标准，除非能够确定以后的工作方式就是“隔空喊话”和“隔空撸码”。

也不推荐把本套题中的内容作为一面，一面应该是作为了解应聘者的职业状况，基础水平（也是就是逻辑能力），推荐本套题作为二面，涵盖基本的iOS开发基础知识，而且时间能够较好的把握在三十分钟内，三面应该侧重项目实际情况，而且最好face to face。四面就HR面了吧，尽量不要有五面了，太难受了。其实三面是最佳的。

记住！不要套题，而应该是触类旁通，引出其它问题。
