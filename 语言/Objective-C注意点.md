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








持续更新中.....
