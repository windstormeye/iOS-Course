---
title: More-DesignPattern
date: 2018-02-02 18:38:51
tags:
---

设计模式，这是一个可以持续投入研究的问题，当初我一直不能理解学长们口中谈论的设计模式到底是什么意思，什么是MVC、MVP、MVVM甚至CDD呢？以及现在层出不穷的MVX等等🙄。有人这么跟我说，“架构，其实是一个设计上的东西，它可以小到类与类之间的一个交互，可以大到不同的模块之间，或者说不同的业务部门之间的交互都可以从架构的层面去理解它。”

好了，说完后我更加懵逼了，这还是没说明白啊。也就一直拖着。随后我开始了第一个自己所谓的“项目”——[“大学+”](https://github.com/windstormeye/CampusPlus)，咱们实话实说，开始大学+之前时间上我有在帮一个学长做他的个人项目一部分，跟我说这个项目整体的架构是MVC，但是当时我哪知道啥是MVC啊，刚开始他丢给我做一个用户登陆模块，我只能依葫芦画瓢，当时根本就不知道啥叫Model，啥叫block，可是当时项目中却充满着大量的Model和block以及各种delegate。😅。迷茫了好几天，最后不管怎么说也是瞎做完了，给学长review的时候居然被他发现了我没用二次封装的AFNetworking网络请求manager，而是自己又搞了一个贼差劲的破东西，被数落了一番后，我当时还是没啥概念，还是不知道为啥要这么做，怎么做。

开始“大学+”项目后，刚开始我同样还是没有拎清楚到底什么是设计模式，导致在项目开展过程中很多模块的实现方式都是乱七八糟，数据源都是瞎给的，甚至有些页面的数据源都重复获取了好几次，但是神奇的地方就在于居然能够把这个项目做完了！！！😅

在前年暑假的重构期间，我在习得了一些设计模式的思想以及大量的实践之后，慢慢的发现！原来我当初的设计是在趋向于MVC的，只不过当时实在是无法hold得住到底什么是MVC才会导致在View中不但做了逻辑还做了model的事情。

随后展开了艰难的重构之路，在重构期间，我又对设计模式有了一个新的认识，开始发现不是某个项目去迎合设计模式、架构，而是设计模式、架构来迎合项目的实际，也就因为是这种情况的出现，最开始软件行业基本上都套用MVC，但是在越来越多的实际开发过程中发现，一昧的死守MVC实际上还有破坏项目实际的耦合，随后才慢慢的衍生出根据不同的开发平台适用的MVP、MVX、MVVM、CDD等。

在我日后的学习和工作中，运用到最多的就是MVC，甚至说基本上都是MVC，毕竟MVC是软件行业的“常青树”，基本上都能够用MVC来构建每一个软件产品，而且MVP、MVVM等可以说都是的MVC的变种，本质上也都还是MVC。

在这篇文章中，我将结合以往学习和工作经验梳理一遍关于耦合、MVC、MVP、MVVM的核心知识点，并编写对应实例进行讲解，也作为自己在设计模式上的理解与总结。

## 架构基础

在讲解三大设计模式之前，我们先来做一些架构基础的工作。之所以要对项目做整体的分层架构设计是因为随着项目进度的展开，日益增长业务逻辑和代码数量远远超出了开发人员所能精确分析掌握的能力。一旦嗅探出项目中有即将“腐烂”的部分，如果再不加以维护日后就一定会变得更加腐烂。🙂

为了防止以上问题的发生，慢慢的萌生出了“软件工程”的科学指导软件开发方法，以工程的思路去规范、进行软件开发工作，同时其衍生品——“设计模式”的思路也慢慢的被广大软件从业者所接受，从此软件行业走进了有科学思想指导的春天！😓。

架构核心是耦合，简单来说耦合可以是两个类之间的交互，当然也可以是三个类甚至更多的类，从大的角度来说，可以是不同的业务模块之间的交互，如何让这些的模块之间的联系或者影响更少，这就我们所说的解耦的概念。

处理好项目中各个类甚至各个模块之间的耦合关系是长久以来软件工程专家甚至开发人员所追求的“至上宝典”，因为产品的不同，其业务流程模型也不同，需要解决的核心问题也不同，围绕其做的架构设计也不能一概而论，而在iOS中解决耦合关系，可以分为三个层次。

1. 直接耦合。双方都知道对方的存在。
2. delegate。只有一方知道对方的存在。
3. notification。双方均不知道对方的存在。

以上三种为架构设计所采用的基本耦合方式，当然还有一些其它的方式，不过这些方式都牵扯到了平台差异性，非iOS端做不到，比如KVO等。以上列举的三种方式具备通俗性，各大平台均可实现。

### 直接耦合

直接耦合做法是最差的一种耦合方式，甚至可以说耦合度最高的一种，类与类或者模块与模块之间互相都知道了双方的存在。当然，这种直接耦合的方式不能说很差，只能说它的用处体现的地方非常局限，不过，大部分同学（包括我自己）在最开始写东西的时候都是“直接耦合”的实践者，它的“简单粗暴”是最吸引人的地方（当然这也是它的致命缺点）

实现“直接耦合”模式需要用到一下场景，Manager发布Task，Worker执行Task，执行Task完成后告诉Manager，Manager庆祝Task完成。因此我们的文件目录结构如下所示，
```shell
|____Worker.m
|____Manager.h
|____Worker.h
|____Manager.m
|____decoupleViewController.h
|____decoupleViewController.m
```

```ObjC
// ----- manager -----
#import "Manager.h"
#import "Worker.h"

@implementation Manager

// 庆祝Task完成
- (void)celebratePrintTask {
    NSLog(@"celebrate Task!");
}

// 发布Task给Worker
- (void)beginPrintTask {
    Worker *woker = [[Worker alloc] init];
    [woker doPrintTask];
}

@end
```

```ObjC
// ----- worker -----
#import "Worker.h"
#import "Manager.h"

@implementation Worker

// 执行Task
- (void)doPrintTask {
    NSLog(@"finish work!");
    
    Manager *manager = [[Manager alloc] init];
    [manager celebratePrintTask];
}

@end
```

而想要把Manage和Worker联系起来，我们得通过decoupleViewController，

```ObjC
#import "decoupleViewController.h"
#import "Manager.h"

@implementation decoupleViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.view.backgroundColor = [UIColor lightGrayColor];
    
    // 创建Manage，让Manage制作Task
    Manager *manager = [[Manager alloc] init];
    [manager beginPrintTask];
}
```

从以上代码中我们可以看到这种直接耦合的写法根本算不上是设计模式，就是一种“随用随写”的风格，缺点大家应该也能看得清楚，以上边代码来说，如果要完成一个Task，Manager是要知道Worker的存在，而且也只能用Worker去完成，而不能让比如说Student去完成。（除非你要再生成一个Student）当然，跟产品的实际设计还是有很大关系的。如果我们要做的东西非常小，或者某个模块比较小，使用这种模式或风格去完成会大大缩小开发成本。

### delegate

delegate，代理设计模式，主要用于反向传值。关于代理的细节在[上一篇文章](http://pjhubs.cn/2018/01/30/More-%E9%A1%B5%E9%9D%A2%E4%BC%A0%E5%80%BC/)中已经做了讲解，如果还是套用Manager和Worker的思路去讲解，使用delega后Worker可以不用管是Manager还是Student甚至是Father去发布的Task，它只管完成。（反过来也可以），因此实际上Worker是不知道manager的存在的，只有manager才知道到底是谁去给他完成了任务。

映射到生活中，这个例子就相当于“我”这个程序员屌丝根本就不管甲方是谁，来活我就做，我相当于worker，甲方可以是BAT，可以是山西煤老板，也可以是美少女战士，这些相当于manager，manager来找到我这个worker，指定我去完成他们的Task。

创建好的文件目录结构为：(跟之前并无区别)
```shell
|____delegateViewController.h
|____delelgateManager.h
|____delegateWoker.h
|____delegateViewController.m
|____delegateWoker.m
|____delelgateManager.m
```

worker的改动为：（相当于指定了工作协议）
```ObjC
// ----- delegateWorker.h -----
#import <Foundation/Foundation.h>

@protocol delegateWorkerDelegate <NSObject>

- (void)donePrintTask;

@end

@interface delegateWoker : NSObject

@property (nonatomic, weak) id<delegateWorkerDelegate> workerDelegate;

- (void)doPrintTask;

@end

// ----- delegateWorker.m -----
#import "delegateWoker.h"

@implementation delegateWoker

- (void)doPrintTask {
    NSLog(@"finish work!");
    
    [_workerDelegate donePrintTask];
}

@end
```
worker变得简单一些，它只管做东西。而manager变为了，

```ObjC
// ----- delegateManager.h -----
#import <Foundation/Foundation.h>

@interface delelgateManager : NSObject

- (void)beginPrintTask;

@end

// ----- delegateManager.m -----
#import "delelgateManager.h"
#import "delegateWoker.h"

@interface delelgateManager () <delegateWorkerDelegate>

@end

@implementation delelgateManager

- (void)beginPrintTask {
    delegateWoker *woker = [[delegateWoker alloc] init];
    woker.workerDelegate = self;
    [woker doPrintTask];
}

- (void)donePrintTask {
    NSLog(@"celebrate Task!");
}
```

从上以上代码中我们可以看到，manager遵守了worker的delegate（相当于给worker的工作协议签了名）并实现了delegate的代理方法（相当于work的工作成果），在donePrintTask的代理方法中可以庆祝Task完成。delegateViewController的代码跟之前是一样的。

```ObjC
#import "delegateViewController.h"
#import "delelgateManager.h"

@implementation delegateViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.view.backgroundColor = [UIColor lightGrayColor];
    
    delelgateManager *manager = [[delelgateManager alloc] init];
    [manager beginPrintTask];
}

@end
```

### notification

通知，这部分内容统一也在[上一篇文章](http://pjhubs.cn/2018/01/30/More-%E9%A1%B5%E9%9D%A2%E4%BC%A0%E5%80%BC/)中做了较为详细的讲解。

使用通知进行架构耦合的文件目录为：
```shell
|____notification.h
|____notifyWorker.h
|____notifyManager.m
|____notificationViewController.m
|____notifyWorker.m
|____notificationViewController.h
|____notifyManager.h
```
因为很多东西在上一篇文章中都已经一一讨论过了，在此不做过多赘述，我们来看使用通知的核心代码，
```ObjC
// ----- Manager -----
#import "notifyManager.h"
#import "notification.h"

@implementation notifyManager

- (instancetype)init {
    self = [super init];
    if (self) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(celebrateWork) name:NOTIFICATION_PRINTTASKDONE object:nil];
    }
    return self;
}

- (void)beginPrintTask {
    [[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_BEGINPRINTTASK object:nil userInfo:nil];
}

- (void)celebrateWork {
    NSLog(@"celebrate work!");
}

@end
```

```ObjC
// ----- worker -----
#import "notifyWorker.h"
#import "notification.h"

@implementation notifyWorker

- (instancetype)init {
    self = [super init];
    if (self) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(doPrintTask) name:NOTIFICATION_BEGINPRINTTASK object:nil];
    }
    return self;
}

- (void)doPrintTask {
    NSLog(@"finish work!");
    [[NSNotificationCenter defaultCenter] postNotificationName:NOTIFICATION_PRINTTASKDONE object:nil];
}

@end
```

从以上两段代码中我们可以看出，manager不知道发布Task后谁去完成，Work不知道完成的Task是谁发布的，也就是双方都不知道是谁，它们只知道如果有人发送了`NOTIFICATION_BEGINPRINTTASK`（开始工作）和`NOTIFICATION_PRINTTASKDONE`（工作结束）的通知后就调用各自的处理方法。

虽然通知看上去好像是解决两个类或模块之间耦合度最低的方法，但同时也是风险较高的一个方法，如果通知管理得不好，debug起来可是一件异常痛苦的事情。😝

以上就是需要大家提前了解的一些架构基础知识，其它的比如MVC、MVP、MVVM等设计模式都需要用到对应的知识，在后续的设计模式讲解过程中会大量保留此类做法。

## MVC

MVC为苹果官方推荐的设计模式，其为**Model-View-Controller**的缩写。简单来说Model就是数据源，访问Model中的属性或者方法即可拿到相应的数据源；View为展示给用户的视图，上边可以堆积入一些button、label或者ImageView等等，并且还负责把从Model中获取到的数据渲染出来；Controller主要做的事情就是搞定Model何时去拉取数据，View何时去加载拉取到的Model以及View的操作何时响应给Model重新拉取数据，需要注意的是，View和Model并无直接联系，View只是有一个Model的属性，View利用该Model属性解析给对应控件进行赋值，它们之间并不能直接操作，如下所示，

<img src="http://7xszq8.com1.z0.glb.clouddn.com/ww%20%289%29.png" width = "60%" height = "60%" align=center />

综上所述就是MVC的核心思想，但实际上不会有人严格遵守这么做的，都是给你瞎搞，这都是摆我大天朝产品经理所赐，某些奇葩需求还真能让你写出来“四不像”（也有可能实力不足？🙄）

举个🌰🍐！！！以下为MVC架构的最小集文件目录，

```shell
|____MVCView.m
|____MVCModel.h
|____MVCModel.m
|____MVCView.h
|____MVCViewController.m
|____MVCViewController.h
```

在MVCView中我们要写明需要加载的控件，其中我们加载了一个UILabel和UIButton，
```ObjC
- (void)initView {
    self.backgroundColor = [UIColor darkGrayColor];
    
    self.tipsLabel = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self addSubview:self.tipsLabel];
    self.tipsLabel.font = [UIFont systemFontOfSize:25];
    self.tipsLabel.textAlignment = NSTextAlignmentCenter;
    
    UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100, 300, 200, 30)];
    [self addSubview:btn];
    [btn addTarget:self action:@selector(btnClick) forControlEvents:1<<6];
    [btn setTitle:@"点我啊！" forState:UIControlStateNormal];
}
```

因为View只负责做视图和数据的展示，其中涉及到数据的逻辑交互都尽量少甚至不要在View的处理，因此我们要把View中的UIButton的点击事件代理出去给Controller进行处理，并且我们的View也是不能自己去拉取数据的，而是应该暴露出一个Model属性供Controller自行调配，因此我们的MVCView.h中可以这么写，

```ObjC
#import <UIKit/UIKit.h>
#import "MVCModel.h"

@protocol MVCViewDelegete <NSObject>

- (void)MVCViewBtnClick;

@end

@interface MVCView : UIView

@property (nonatomic, strong) MVCModel *model;
@property (nonatomic, weak) id<MVCViewDelegete> viewDelegate;

@end
```

而完整的MVCView.m我们可以把对应的逻辑补充完整，

```ObjC
#import "MVCView.h"

@interface MVCView ()

@property (nonatomic, strong) UILabel* tipsLabel;

@end

@implementation MVCView

- (id)init {
    self = [super init];
    if (self) {
        [self initView];
    }
    return self;
}

- (void)initView {
    self.backgroundColor = [UIColor darkGrayColor];
    
    self.tipsLabel = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self addSubview:self.tipsLabel];
    self.tipsLabel.font = [UIFont systemFontOfSize:25];
    self.tipsLabel.textAlignment = NSTextAlignmentCenter;
    
    UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100, 300, 200, 30)];
    [self addSubview:btn];
    [btn addTarget:self action:@selector(btnClick) forControlEvents:1<<6];
    [btn setTitle:@"点我啊！" forState:UIControlStateNormal];
}

- (void)setModel:(MVCModel *)model {
    _model = model;
    self.tipsLabel.text = model.contentString;
}

- (void)btnClick {
    if (_viewDelegate) {
        [_viewDelegate MVCViewBtnClick];
    }
}

@end
```

从以上MVCView.m代码中我们可以看到，重写了Model的setter方法，拿到model后我们再接着给对应的label赋值数据源即可，在button对应的点击事件中代理出去，

在MVCController部分，我们不但要把MVCView和MVCModel的关系都确认联系起来，还要明确这两者何时进行交互（例子可能不够复杂，并不能体现出何时进行交互🙂）

```ObjC
#import "MVCViewController.h"
#import "MVCView.h"
#import "MVCModel.h"

@interface MVCViewController () <MVCViewDelegete>

@property (nonatomic, strong) MVCModel* model;
@property (nonatomic, strong) MVCView* MVCView;

@end

@implementation MVCViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.view.backgroundColor = [UIColor lightGrayColor];
    
    self.model = [[MVCModel alloc] init];
    self.model.contentString = @"MVC model";
    
    self.MVCView = [[MVCView alloc] init];
    self.MVCView.frame = self.view.bounds;
    self.MVCView.model = self.model;
    self.MVCView.viewDelegate = self;
    [self.view addSubview:self.MVCView];
    
}

- (void)MVCViewBtnClick {
    NSInteger interger = random() % 10;
    self.model.contentString = [NSString stringWithFormat:@"%ld", (long)interger];
    self.MVCView.model = self.model;
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}
```

而我们的MCVModel，因为我们只需要的对MVCView上的Label进行文本的替换，因此我们的model实体只需要一个NSString属性即可。

```ObjC
// -----  MVCModel.h -----
#import <Foundation/Foundation.h>

@interface MVCModel : NSObject

@property (nonatomic, copy) NSString* contentString;

@end

// -----  MVCModel.m -----

#import "MVCModel.h"

@implementation MVCModel

@end
```

通过以上操作，我们即可完成MVC设计模式的最小集设计，大家应该能够对MVC有一个初步的认识，MVC架构做到最后会导致C层非常庞大，甚至四五千行代码都是有可能的。

<img src="http://7xszq8.com1.z0.glb.clouddn.com/Feb-03-2018%2011-07-24.gif" width = "40%" height = "40%" align=center />


## MVP

MVP的全称为Model-View-Presenter，可以看到缺少了Controller，替换成了Presenter。但是不管怎么说其本质还是MVC的分层架构的思想，只不过把Controller的要做的事情降低了。

不过在iOS中并不推荐使用MVP用于项目架构，因为在iOS中是“原生支持”MVC，导致如果我们硬是要用上MVP，整体的项目文件目录就变成了：

```shell
|____MVPModel.h
|____MVPModel.m
|____MVPView.h
|____MVPView.m
|____Presenter.h
|____Presenter.m
|____MVPViewController.h
|____MVPViewController.m
```

我们还是要创建出来Controller，只不过这里的Controller可以认为是当前该MVP模块的容器（因为在iOS中就是用各种Controller联系起来的😓。），我们先来看看此时的Controller做了哪些事情，

```ObjC
#import "MVPViewController.h"
#import "Presenter.h"
#import "MVPView.h"
#import "MVPModel.h"

@interface MVPViewController ()

@property (nonatomic, strong) MVPView*    mvpView;
@property (nonatomic, strong) MVPModel*    mvpModel;
@property (nonatomic, strong) Presenter*    presenter;

@end

@implementation MVPViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self initView];
}

- (void)initView {
    self.view.backgroundColor = [UIColor lightGrayColor];
    
    self.presenter = [Presenter new];
    
    self.mvpView = [MVPView new];
    self.mvpView.frame = self.view.bounds;
    [self.view addSubview:self.mvpView];
    self.mvpView.viewDelegate = self.presenter;
    
    self.mvpModel = [MVPModel new];
    
    self.presenter.mvpModel = self.mvpModel;
    self.presenter.mvpView = self.mvpView;
    self.mvpModel.contentString = @"2333";
    [self.presenter doPrintWork];
}

@end
```

可以看到，实际上Controller的用处只是把Model-View-Presenter这三个东西联系起来而已，逻辑都在Presenter里，

```ObjC
// ----- Presenter.h -----
#import <Foundation/Foundation.h>
#import "MVPModel.h"
#import "MVPView.h"

@interface Presenter : NSObject <MVPViewDelegete>

@property (nonatomic, strong) MVPModel*    mvpModel;
@property (nonatomic, strong) MVPView*    mvpView;

- (void)doPrintWork;

@end

// ----- Presenter.m -----

#import "Presenter.h"

@implementation Presenter

- (void)doPrintWork {
    NSString *content = self.mvpModel.contentString;
    self.mvpView.content = content;
}

- (void)MVPViewBtnClick {
    NSInteger interger = random() % 10;
    self.mvpModel.contentString = [NSString stringWithFormat:@"%ld", (long)interger];
    self.mvpView.content = self.mvpModel.contentString;
}

@end

```

MVPView和MVCView有一个不一样的地方，

```ObjC
// ----- MVPView.h -----
#import <UIKit/UIKit.h>
#import "MVPModel.h"

@protocol MVPViewDelegete <NSObject>

- (void)MVPViewBtnClick;

@end

@interface MVPView : UIView

@property (nonatomic, strong) NSString*    content;
@property (nonatomic, weak) id<MVPViewDelegete>    viewDelegate;

@end

// ----- MVCView.h -----
#import <UIKit/UIKit.h>
#import "MVCModel.h"

@protocol MVCViewDelegete <NSObject>

- (void)MVCViewBtnClick;

@end

@interface MVCView : UIView

@property (nonatomic, strong) MVCModel *model;
@property (nonatomic, weak) id<MVCViewDelegete> viewDelegate;

@end

```

我们可以看到，在MVC模式中的View的数据源是Model类型，而在MVP中的View是不知道Model的类型，只知道View需要什么数据（可以是任意基本数据类型NSString、NSDictionary等），而不管Model。因此可以有个初步的感受，MVC中的View和Model有跟隐含的虚线连接着，View是知道Model的，而在MVP中除了Presenter外，View和Model都是互相不知道的，可以说这是又进一步的把耦合度减低了。

综上所述，实际上MVP在iOS中并不适用，也可以说我不喜欢，可能写的实例还没体现出来我为什么不喜欢MVP的实际原因，因为不管怎么搞你总是会拉着一个拖油瓶Controller，MVP的核心思想是用Presenter去替代Controller，让View和Model之间的联系完全取消，但是我们无法改变Controller在iOS中的地位😓，反而MVP在Android中会大放异彩，因为在Android中没有像在iOS中“万事皆需Controller”的概念。


## MVVM

终于到了MVVM这个我最喜欢的架构了😝。MVVM全称为Model-View-ViewModel，同时也是基于MVC的延伸品，只不过它没MVP那般强硬，使用MVVM我们只需要记住一个思想——“双向绑定”，我们只要达到View和ViewModel、Model和ViewModel的双向绑定即可。

不需要管是否有Controller的存在，而且MVVM也不允许View和Model直接联系，而是通过一个ViewModel实例去联系起来，而且这个ViewModel还是和View与Model进行了双向绑定的，只要Model中的数据发生了改变，View就会监听到这个改变，从而赋值达到重新渲染数据刷新UI。

所以我们要解决的就是如何进行“双向绑定”，而这个“双向绑定”只是个指导思想，我们完全可以用前文“架构基础”中讲述的三个方法完成，而之前我一直觉得使用苹果自己提供的KVO（key——value-Observe）写起来太累了就只用了delegate去实现，当然也可能是因为团队小伙伴们对MVVM跟我当初一样比较迷茫，再加上我偷懒把ViewModel揉在Controller里，使用了delegate来实现“双向绑定”，就导致了大家看得云里雾里。😂。

刚好在这段时间中有网友推荐使用Facebook开源的KVOController能够有效降低手撸原生KVO API的痛苦（我是觉得很痛苦），借此机会我们来举个KVO实现MVVM最小集的🌰🍐。

### MVVMController

同样Controller也是要完成Model和View关系建立

```ObjC
#import "MVVMViewController.h"
#import "MVVMViewModel.h"
#import "MVVMModel.h"
#import "MVVMView.h"

@interface MVVMViewController ()

@property (nonatomic, strong) MVVMViewModel*    viewModel;
@property (nonatomic, strong) MVVMView*    mvvmView;
@property (nonatomic, strong) MVVMModel*    mvvmModel;

@end

@implementation MVVMViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.mvvmView = [[MVVMView alloc] init];
    self.mvvmView.frame = self.view.bounds;
    [self.view addSubview:self.mvvmView];
    
    self.mvvmModel = [[MVVMModel alloc] init];
    self.mvvmModel.content = @"2333";
    
    self.viewModel = [[MVVMViewModel alloc] init];
    self.viewModel.contentString = self.mvvmModel.content;
    
    [self.mvvmView setWithViewModel:self.viewModel];
    [self.viewModel setWithModel:self.mvvmModel];
}

@end
```

### MVVMView

```ObjC
// ----- MVVMView.h -----
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>
#import "MVVMViewModel.h"

@interface MVVMView : UIView

@property (nonatomic, strong) NSString*    content;

- (void)setWithViewModel:(MVVMViewModel *)vm;

@end

// ----- MVVMView.m -----
#import "MVVMView.h"
#import "FBKVOController.h"
#import "MVVMViewModel.h"
#import "NSObject+FBKVOController.h"

@interface MVVMView ()

@property (nonatomic, strong) UILabel*    tipsLabel;
@property (nonatomic, strong) MVVMViewModel*    vm;

@end

@implementation MVVMView

- (instancetype)init {
    self = [super init];
    if (self) {
        [self initView];
    }
    return self;
}

- (void)initView {
    self.backgroundColor = [UIColor lightGrayColor];
    
    self.tipsLabel = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self addSubview:self.tipsLabel];
    self.tipsLabel.font = [UIFont systemFontOfSize:25];
    self.tipsLabel.textAlignment = NSTextAlignmentCenter;
    
    UIButton *btn = [[UIButton alloc] initWithFrame:CGRectMake(100, 300, 200, 30)];
    [self addSubview:btn];
    [btn addTarget:self action:@selector(btnClick) forControlEvents:1<<6];
    [btn setTitle:@"点我啊！" forState:UIControlStateNormal];
}

- (void)setContent:(NSString *)content {
    _content = content;
    self.tipsLabel.text = content;
}

- (void)btnClick {
    [self.vm doPrintWork];
}

- (void)setWithViewModel:(MVVMViewModel *)vm {
    self.vm = vm;
    
    [self.KVOController observe:vm keyPath:@"contentString" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld block:^(id  _Nullable observer, id  _Nonnull object, NSDictionary<NSKeyValueChangeKey,id> * _Nonnull change) {
        NSString *newContent = change[NSKeyValueChangeNewKey];
        self.tipsLabel.text = newContent;
    }];
}
```

MVVMView中，我们使用了Facebook开源的KVOController封装好的苹果提供的原生KVO API，MVVMView的其它东西跟之前一样，只不过它的数据源获取方法变成了，
```ObjC
- (void)setWithViewModel:(MVVMViewModel *)vm {
    self.vm = vm;
    
    [self.KVOController observe:vm keyPath:@"contentString" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld block:^(id  _Nullable observer, id  _Nonnull object, NSDictionary<NSKeyValueChangeKey,id> * _Nonnull change) {
        NSString *newContent = change[NSKeyValueChangeNewKey];
        self.tipsLabel.text = newContent;
    }];
}
```

它在此使用KVO监听了vm对象的contentString属性，只要我们把MVVMView和MVVMViewModel的对应关系都确定了，当contentString发生变化时，能够实时的修改数据刷新UI。当然，如果不想达到这种自动的效果，那就跟我当初一样用delegate去手动实现“双向绑定”吧🙂。

### MVVMViewModel

```ObjC
// ----- MVVMViewModel -----
#import <Foundation/Foundation.h>
#import "MVVMModel.h"

@interface MVVMViewModel : NSObject

@property (nonatomic, strong) NSString*    contentString;

- (void)setWithModel:(MVVMModel *)model;
- (void)doPrintWork;

@end

// ----- MVVMViewModel -----
#import "MVVMViewModel.h"

@interface MVVMViewModel ()

@property (nonatomic, strong) MVVMModel*    mvvmModel;

@end

@implementation MVVMViewModel

- (instancetype)init {
    self = [super init];
    if (self) {
        
    }
    return self;
}

-(void)setWithModel:(MVVMModel *)model {
    self.mvvmModel = model;
    self.contentString = model.content;
}

- (void)doPrintWork {
    NSInteger interger = random() % 10;
    self.mvvmModel.content = [NSString stringWithFormat:@"%ld", (long)interger];
    self.contentString = self.mvvmModel.content;
}

@end

```

其它类都是一样的。

---
