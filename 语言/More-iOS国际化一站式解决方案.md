---
title: More-iOS国际化一站式解决方案
date: 2018-04-10 21:10:40
tags:
- iOS
- 国际化
---

关于iOS开发中的国际化（也可称为多语言）在网上的文章多如牛毛，不过总结起来就那么一回事，不是说他们写的不好我写的多好，而是说过于零散。

现在，我将结合实际场景需求进行国际化做法详解。可以肯定的是，Android的国际化做法大同小异，无非也就是各个语言版本的文件替换，我们先来分析下真实的需求是怎么一回事。

## 国际化需求：
1. 只提供English和Chinese Simplified两种语言；
2. App名称跟随系统语言变化；
2. 用户首次打开app时，app的语言与系统语言保持一致（系统语言为非简体中文，默认app都是英文）用户手动更改语言之后，之后都记忆用户选择的语言；
3. 用户在App内切换语言后，App本身所有文本信息全部替换成对应语言。

根据需求，我比较纠结的地方是，App的静态文本数据可以存两份在本地，也就是English一份Chinese Simplified一份，但请求的API是同时返回两份中英文数据or分中英文两个接口？如果是要一个接口同时返回了中英文两份数据，显然会加大数据包的大小，其次用户很有可能从安装App的那天开始就不再切换App语言，甚至平均几个星期才换一次，同时返回两份数据是否多余，但是这么做几乎可以达到“无感知”数据源切换，相当于是说，一旦用户选择好了要切换语言，“啪嗒”点了完成，立马pop掉当前页面，然后整个App的数据源中英文切换可以几乎用“瞬间完成”来形容。

如果是分中英文两个接口，实际上就会出现微信在进行语言切换时的loading菊花，因为要重新拉取英文版数据，不过好处是可以减少上一种做法的数据包整体大小。这两种做法我都有实践过，如果你的App是非常固定，不会频繁出现语言切换的需求，那么可以使用第二种；如果App有一天之内可能会频繁切换多次语言的情况，第一种无疑。

经过一番探讨，虽然要供给拉美、北美和欧洲的同学使用，但是不会出现频繁切换语言的情况，所以，最终我们选择了第二种解决方案。先来看一张最终成果gif图，

<img src="https://i.loli.net/2018/04/10/5accbf4011d4d.gif" width = "40%" height = "40%" align=center />


从上图中可以看到其实并没有对数据源进行切换，因为。。。。后台没写完😓。

不过也不影响我们的讲解，首先，明确一个概念，我们能够做的国际化语言支持iOS系统中自带的所有语言，只要你能在系统设置中找到的语言，就能够对你的App做对应版本的国际化适配；其次，每对App适配一种语言，就要单创建出一个语言文件（要不然会引起冲突）。OK，我们正式进入讲解。


### 首先创建一个工程
我起名为`languageTest`。


### 工程初始化
```objc
// AppDelegate.m

#import "navOneViewController.h"
#import "navTwoViewController.h"

@interface AppDelegate ()

@property (nonatomic, strong) UITabBarController *rootTabBar;

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
    self.rootTabBar = [[UITabBarController alloc]init];
    self.rootTabBar.delegate = (id)self;
    self.window.rootViewController = self.rootTabBar;
    [[UITabBar appearance] setBarTintColor:[UIColor whiteColor]];

    navOneViewController *navOneController = [navOneViewController new];
    UINavigationController *nav1 = [[UINavigationController alloc] initWithRootViewController:navOneController];
    nav1.title = @"首页";

    navTwoViewController *navTwoController = [navTwoViewController new];
    UINavigationController *nav2 = [[UINavigationController alloc] initWithRootViewController:navTwoController];
    nav2.title = @"发现";

    self.rootTabBar.viewControllers = @[nav1, nav2];

    [self.window makeKeyAndVisible];

    return YES;
}

```

在`Appdelegate`中，我们创建一个具备基本展示功能的tabBar及挂载在其之上的VC，

```objc
// navOneViewController.m

- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor blueColor];

    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self.view addSubview:label];
    label.font = [UIFont systemFontOfSize:25];
    label.textColor = [UIColor whiteColor];
    label.text = @"这是首页";

}


// navTwoViewController.m

- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor orangeColor];

    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self.view addSubview:label];
    label.font = [UIFont systemFontOfSize:25];
    label.textColor = [UIColor whiteColor];
    label.text = @"这是发现";

    UIButton *button = [[UIButton alloc] initWithFrame:CGRectMake(100, 300, 100, 100)];
    [self.view addSubview:button];
    [button addTarget:self action:@selector(buttonClick) forControlEvents:UIControlEventTouchUpInside];
    button.backgroundColor = [UIColor blueColor];
    [button setTitle:@"改变语言" forState:UIControlStateNormal];

}

- (void)buttonClick {

}

```

在各自对应的VC中写下相关UI，并预留相关Button点击事件即可。


### 创建语言文件

创建路径： file -> new -> file... -> String File，文件名严格命名为——“Localizable”，创建好该文件后，点击该文件，并打开Xcode的右边功能区（不知道应该叫啥），在Localization功能区勾选语言版本，如果此时你并未看到或者只有English可选，我们需要到PROJECT -> info -> Localization，添加需要的语言。

添加完成后，会在之前创建的Localization.string文件下看到多出来的语言文件，我选择了English和Chinese Simplified。现在，我们已经可以在对应生成的语言文件中进行需要多语言替换的字段编写了。

```objc
// Localizable.string/English文件

"home" = "home";
"homeString" = "I'm home";

"discover" = "discover";
"discoverString" = "I'm discover";

"change" = "change";

// Localizable.string/Chinese(Simplified)文件

"home" = "首页";
"homeString" = "我是首页";

"discover" = "发现";
"discoverString" = "我是发现";

"change" = "改变语言";
```


并新建一个pch文件，pch文件同样也是头文件，不过这是一个特殊头文件，是一个预编译文件，位于该文件中的所有内容，能够被其他所有源文件共享和访问，相信你也看出来了，如果在pch文件中写了大量的不是必须文件，则会延长编译期时间，我们可以在.pch文件中放：
1. 全局宏；
2. 整个工程中都能用上的头文件；
3. 动态更加当前App运行的环境切换相关宏（debug or release）。

因此，我们需要创建一个pch文件去存放接下来要在整个工程中都要用到的判断语言环境的中英文宏。创建一个pch文件的方式为，file -> new -> file... -> 搜“pch”关键字，创建它。

进入工程配置 -> TARGET -> Build Settings -> 搜pch关键词 -> 在“Apple LLVM 9.0 - Language”下的Prefix Header中，双击输入你的.pch文件路径，我写的是`$(SRCROOT)/PrefixHeader.pch`，填写完毕，回车，会看到生成的绝对路径，确定pch文件路径是否正确。一切都没问题后，编译通过即可。

```
在pch文件中，写入以下宏定义，
```objc
#define AppLanguage @"appLanguage"
#define PJLocalString(key) \
[[NSBundle bundleWithPath:[[NSBundle mainBundle] pathForResource:[NSString stringWithFormat:@"%@",[[NSUserDefaults standardUserDefaults] objectForKey:@"appLanguage"]] ofType:@"lproj"]] localizedStringForKey:(key) value:@"" table:nil]
```

首先定义了一个`AppLanguage`宏，推荐大家的命名更加多样化一些，因为OC并没有namespace，如果我们的命名过于简单，就会导致和Apple本身自定义的NSUserDefaults默认值产生冲突。

`PJLocalString(key)`这个宏“定义”了一个更长的方法，我们也都明确了一个概念，在iOS中的每个国际化语言，就对应着一个文件，这个文件就保存在App沙盒的根目录中，我们要做的就是在某个时机替换系统所采用的语言文件即可，而`PJLocalString(key)`这个宏所做的事情，就是替换！先从NSUserDefaults中取出对应的语言key（en还是zh-Hans），根据语言key去索引到对应的.lproj文件，最后把要替换的关键词传入，抛出找到的对应值（我觉得找的这个过程用的结构应该不是hashmap，真的很快。😨）


多语言文件有了，宏也有了，那怎么用呢？举个例子！

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor blueColor];

    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self.view addSubview:label];
    label.font = [UIFont systemFontOfSize:25];
    label.textColor = [UIColor whiteColor];
    label.text = PJLocalString(@"homeString");

}
```

只需要在多语言文字的地方调用`PJLocalString()`宏，传入对应key即可。但是此时运行工程，会发现啥都没了，是因为我们并未对NSUserDefaults中做当前语言的设置，这就导致了取出的值为nil。所以，还需要在`AppDelegate`文件中设置初始语言，
```objc
if(![[NSUserDefaults standardUserDefaults] objectForKey:AppLanguage]){
        [[NSUserDefaults standardUserDefaults] setObject:@"zh-Hans" forKey:AppLanguage];
        [[NSUserDefaults standardUserDefaults] synchronize];
    }
```

这样，我们即可完成第一次进入App时初始化基础语言，如果我们想要实时更改呢？这就需要用到了通知，使用通知机制去给监听语言设置改变的监听者进行相应的处理，

```objc
// 给Appdelegate.m更新以下方法

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    if(![[NSUserDefaults standardUserDefaults] objectForKey:AppLanguage]){
        [[NSUserDefaults standardUserDefaults] setObject:@"zh-Hans" forKey:AppLanguage];
        [[NSUserDefaults standardUserDefaults] synchronize];
    }

    self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
    self.rootTabBar = [[UITabBarController alloc]init];
    self.rootTabBar.delegate = (id)self;
    self.window.rootViewController = self.rootTabBar;
    [[UITabBar appearance] setBarTintColor:[UIColor whiteColor]];

    navOneViewController *navOneController = [navOneViewController new];
    UINavigationController *nav1 = [[UINavigationController alloc] initWithRootViewController:navOneController];
    nav1.title = PJLocalString(@"home");

    navTwoViewController *navTwoController = [navTwoViewController new];
    UINavigationController *nav2 = [[UINavigationController alloc] initWithRootViewController:navTwoController];
    nav2.title = PJLocalString(@"discover");

    self.rootTabBar.viewControllers = @[nav1, nav2];

    // 新增监听方法
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(changeLanguage:) name:@"changeLanguage" object:nil];

    [self.window makeKeyAndVisible];

    return YES;
}

- (void)changeLanguage:(NSNotification *)notify {
    self.rootTabBar.viewControllers[0].tabBarItem.title = PJLocalString(@"home");
    self.rootTabBar.viewControllers[1].tabBarItem.title = PJLocalString(@"discover");
}


// navOneViewController.m更新以下方法

- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor blueColor];

    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self.view addSubview:label];
    label.font = [UIFont systemFontOfSize:25];
    label.textColor = [UIColor whiteColor];
    label.text = PJLocalString(@"homeString");

    // 新增监听方法
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(changeLanguage:) name:@"changeLanguage" object:nil];
}

- (void)changeLanguage:(NSNotification *)notify {
    self.label.text = PJLocalString(@"discoverString");
}


// navTwoViewController.m更新以下方法

- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor orangeColor];

    self.label = [[UILabel alloc] initWithFrame:CGRectMake(100, 100, 200, 20)];
    [self.view addSubview:self.label];
    self.label.font = [UIFont systemFontOfSize:25];
    self.label.textColor = [UIColor whiteColor];
    self.label.text = PJLocalString(@"discoverString");

    self.button = [[UIButton alloc] initWithFrame:CGRectMake(100, 300, 100, 100)];
    [self.view addSubview:self.button];
    [self.button addTarget:self action:@selector(buttonClick) forControlEvents:UIControlEventTouchUpInside];
    self.button.backgroundColor = [UIColor blueColor];
    [self.button setTitle:PJLocalString(@"change") forState:UIControlStateNormal];

    // 新增监听方法
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(changeLanguage:) name:@"changeLanguage" object:nil];
}

- (void)changeLanguage:(NSNotification *)notify {
    self.label.text = PJLocalString(@"discoverString");
    [self.button setTitle:PJLocalString(@"change") forState:UIControlStateNormal];
}

- (void)buttonClick {
    [[NSUserDefaults standardUserDefaults] setObject:@"zh-Hans" forKey:AppLanguage];
    [[NSUserDefaults standardUserDefaults] synchronize];

    // 同步完NSUserDefault后，发送语言更改通知
    [[NSNotificationCenter defaultCenter] postNotificationName:@"changeLanguage" object:nil];
}
```

编译运行吧，见证奇迹的时刻到了~点击“更改语言”button，怎么样，是不是瞬间全都改过了。。😝


但是现在只完成了第一和第四个需求，我们接着来完成第三个需求，“用户首次打开app时，app的语言与系统语言保持一致（系统语言为非简体中文，默认app都是英文）用户手动更改语言之后，之后都记忆用户选择的语言”。

分析一下，该需求的重点在于用户第一次打开App时整体App语言设置跟随系统语言设置，非简体中文之外的语言，都设置成英文，因此，我们需要对`AppDelegate.m`文件进行改造，

```objc

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // App第一次启动跟随系统语言设置
    if(![[NSUserDefaults standardUserDefaults] boolForKey:@"firstLaunch"]){
        [[NSUserDefaults standardUserDefaults] setBool:YES forKey:@"firstLaunch"];
        NSArray *allLanguages = [[NSUserDefaults standardUserDefaults] objectForKey:@"AppleLanguages"];
        NSString *preferredLanguage = allLanguages[0];
        if([preferredLanguage rangeOfString:@"zh-Hans"].location != NSNotFound) {
            [[NSUserDefaults standardUserDefaults] setObject:@"zh-Hans" forKey:AppLanguage];
        } else {
            [[NSUserDefaults standardUserDefaults] setObject:@"en" forKey:AppLanguage];
        }
    }

    ........

```

接下来完成最后一个需求，把App的名称也做国际化适配，如果你之前有过在`info.plist`文件中修改过App的名字，我们现在要做的事情同样也是改名字，而且是针对`info.plist`整个文件做国际化，同样新建一个`string file`文件，命名严格填写为`infoPlist.strings`，并且在Xcode的右边拓展栏中选择`Localizable`，点击生成English和Chinese Simplified多语言文件
```objc
// 在English中写下
CFBundleName = "your english name";
CFBundleDisplayName = "your english name";


// 在Chinese Simplified中写下
CFBundleName = "你的中文名";
CFBundleDisplayName = "你的中文名";

```

OK，以上就是本篇文章所要表达的所有内容，当然这些都是demo级别的code，如果此文对你有帮助，记得对其进行多多改造！

demo地址：
[https://github.com/windstormeye/iOSMorePractices/tree/master/languageTest](https://github.com/windstormeye/iOSMorePractices/tree/master/languageTest)

原文链接：[pjhubs.com](http://pjhubs.com/2018/04/10/More-iOS%E5%9B%BD%E9%99%85%E5%8C%96%E4%B8%80%E7%AB%99%E5%BC%8F%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)
