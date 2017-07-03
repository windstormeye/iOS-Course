# Objective-C

Objective-C是借鉴Smalltalk这门面向对象编程语言的始祖的设计和具备强大的移植性的C，很多时候，我们会把Objective-C写成ObjC、Obj-C或OC。Objective-C是一门比较老的语言了，从1980年，布莱德·考克斯\\(Brad Cox\\)在其公司Stepstone发明Objective-C了它，之后授权给了被赶出苹果公司的乔布斯新创立的NeXT公司，NeXT可以发布自己的Objective-C Compiler和libraries。同时使用Objective-C开发了一套NeXTSTEP，并创建了NeXTSTEP Toolkit软件包，这个工具包用于开发用户界面，功能强大。

我们直接来看一段Objective-C的代码，

![](/assets/123.png)

我们一步一步从头开始细细剖析这段代码，对Objective-C有个初步的了解。

**1、\#import &lt;Foundation/Foundation.h&gt;**

如果你有学过C语言或者java语言，那么你对这段代码的格式一定不会陌生。\#import，就是导入一个框架或者导入一个系统自带的头文件。Foundation，翻译过来就是基础。基础框架内容，与C语言中的stdio.h类似，提供一些例如处理输入输出功能等方法。

**2、int main \(int argc, constchar\* argv\[\]\)**

这段代码的意思大家都知道，main是个函数，argc（argument count：参数个数）是命令行总的参数个数。argv\[\]\(argument vector:指针数组，指向参数内容\)，其中第0个参数是程序的全名，以后的参数命令行后面跟的用户输入的参数。

**3、NSAutoreleasePool \* pool = \[\[NSAutoreleasePool alloc\] init\];**

**\[pool drain\];**

这是iOS 5.0之前的写法，C++是需要手动内存管理，java有垃圾自动回收机制，那么C的超级Objective-C呢？刚开始Objective-C确实是跟C++一样，需要我们手动的去管理内存，但是经过一段时间的发展后，手动管理内存实际上是在浪费时间，因此，在iOS 5.0时代，苹果引入了ARC（Auto Release Pool）自动释放池技术，从而取代MRC（MannulReference Counting）手动引用技术。经过iOS 5后写法变成了

@autoreleasePool{

}

**4、NSLog\(@"Hello, World!"\);**

NSLog是OC中的输出语句，与C语言中的printf函数用法类似，但是比printf强大很多。可能你会很奇怪，为什么感觉每个OC的关键字前都放了一个NS呢？这部分内容啊，大家可以看看乔布斯传，感觉有点形式主义了。因为乔布斯被赶出苹果公司创建了NeXT公司后，得到了Objective-C的授权，可以修改部分内容，所以把每个关键字前都加上了NS。以此代表此版本的Objective-C为NeXT公司所有。



---



以上就是关于Objective-C的部分基础内容。给大家一个进行一个快速入门，关于Objective-C更加细致的内容和学习，大家可以参考文末所列出的链接，因为Objective-C经过这么多年的发展，网上的资料非常的多，笔者在此列出当初学习时所用到的资料，供大家参考。

1、[Objective-C语言学习](https://pan.baidu.com/s/1pLlu8C7)

2、[Objective-C的Foundation框架学习](https://pan.baidu.com/s/1jIoS8J0)

3、这是一个简单的Objective-C小实验，大家可以跟着走一走（其实基础部分的东西大家都是类似的）

