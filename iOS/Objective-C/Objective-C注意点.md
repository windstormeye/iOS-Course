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
