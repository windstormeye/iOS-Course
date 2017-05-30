# Xcode，我来啦！

![](/assets/1.jpg)

Xcode，前身为乔帮主创建NeXT公司时开发的Project Builder，NeXT被苹果公司收购后于2003年正式发布Xcode 1.0。在Xcode出现之前开发Mac OS X应用程序的开发工具很凌乱，直到Xcode 3.0的发布，它成为了开发人员建立 Mac OS X 应用程序的最快捷方式，也是利用新的苹果电脑公司技术的最简单的途径。到3.1版本后，其加入了iOS SDK，这样开发者从此可以不用再单独下载iOS SDK了。

关于SDK是什么，直译过来就是一个“软件开发包”，这里边提供一堆与iOS有关的各种方法接口（也叫API），可以在通过调用这个SDK里提供的各种方法来实现我们在开发过程中的功能。

在4.0版本之前，如果我们想要下载Xcode进行Mac OS X或者iOS系统下的应用程序开发的话，我们必须是开发者才行。4.0版本到来后，苹果公司不单向个人开发者开放了下载权限，非开发者也可以进行下载，不过还得先支付4.99美元。到了4.1版本，Xcode开始面向Mac OS X10.6和10.7版本的所有用户提供免费下载。这种有点“区别”对待的情况，直到4.5版本，Xcode正式提供免费下载。

以上就是Xcode一些简单的背景介绍，目前最新的Xcode版本已经到了8.3.2，可以说Xcode正在变得越来越好，对开发者来说提供了非常多的帮助。接下来，我们将从Xcode的首页开始一步一步与大家一起探索Xcode。

 ![](/assets/Xcode.png)上图为Xcode启动完毕后出现的首页界面。在这个页面中，我们看到了有“Get start with a playground” 、“Create a new Xcode project”和“Check out an existing project”三个选项，笼统的来说，“Get start with a playground”的作用有以下几点：

1、让你可以快速的玩玩Swift；

2、快速的验证你的算法思想，这一点在刷各种编程题的时候用Swift来解决非常的舒服；

3、享受及时编译带来的快感。

我们来看看这个开发利剑 —— “Swift playground”是怎么一回事吧，![](/assets/Swift playground.png)在这里，我们设置了Name为默认的“Myplayground”，平台选择为iOS。选择playground文件存放的路径，这一步就完成了。![](/assets/Swiftplayground1.png)上图中所示的为Swift playground默认创建好的一段代码，这段代码的含义已经很清晰了，第三行告诉我们import，也就是导入了一个东西，这个东西叫做UIKit。第五行的整体意识是创建了一个变量，变量的内容是一个字符串，字符串的内容是"Hello, playground"。var就是variable变量的简写，与之对应的是let，Swift在这一块内容中据说是借鉴了Scala语言的var和val。变量的名字叫做str，这个名字大家可以随意选取。

既然，我们现在知道了str是个变量，那么还可以玩一些这样的事情，![](/assets/Swiftplayground.png)甚至，你还可以在Swift playground中做这样的事情，![](/assets/import.png)你完全可以在Swift playground中以图形化的方式显示出某个值的变化趋势，对使用Swift语言来调试算法的同学来说是一个莫大的帮助。当然，关于Swift playground还有一堆很好玩的功能，笔者在此把大家领进门，剩下的就靠同学们自己努力了。关于Swift playground，笔者就介绍到这里。苹果推出Swift playground主要目的就是让大家快速的学习Swift语法，用Swift搞点事情。



接下来，我们来看看第二个选项 —— “Create a new Xcode project”，创建一个Xcode工程



