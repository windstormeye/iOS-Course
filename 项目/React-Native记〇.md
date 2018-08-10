emmm，这是我今年的第三件事。微信小程序的学习告一段落，开始正式进入RN的学习，RN到现在发展了将近三年，业界有对其有非常大的赞誉，而且现目前不管是整体架构成熟度还是社区的活跃度都非常靠前，可以说是杀入的好时机（其实已经很晚了。）正适合我这种吃不了第一口肉的人。

之所以要学习RN有这么几点原因，其一是有即将开始的项目；其二同样是观望了RN很久，一直很想投身学习；其三就是拓展自己的技能树了。🙂。这次折腾了一下RN的开发环境，刚开始以为会非常顺利，一步到位，实际上却把自己坑得不浅，在此记录一番，给各位也想杀入RN的同学提个醒，不要把时间再次浪费。

在正式进入开发之前，需要我们本地环境上已经安装好Node.js，而安装Node.js有两种方式，你可以去[官网](https://nodejs.org/en/)上下载安装包，直接点击解压出的pkg即可，另外一个方式通过Homebrew（Mac），Homebrew是一款Mac下的一款非常强大软件包管理工具，如果之前没安装过的话，可以通过这行原汁原味的官方安装方法搬运，

```shell
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
安装好Homebrew后，我们通过Homebrew来一步下载node，
```shell
$ brew install node
```

RN官网上是这么写的，但我下载下来的node中并没有顺便也把npm也给一块儿下了（npm是管理node.js的包管理工具），可能每个人的环境不一样？还是我下的node版本太低，没自带npm，如果你发现通过直接install node下载好的node跟我一样也没有npm，先卸载掉之前安装的npm，

```shell
brew uninstall node
```

随后再补充参数，

```shell
$ brew install node --with -npm
```

当然，如果你是从官网上下载手动安装的话，就不用管这个问题了。此时可以执行**node -v和npm -v**来查看这两个东西是否都安装完毕。

接着，我们再通过npm来安装react-native，

```shell
$ npm install -g react-native-cli
```

官网上说，如果你在安装过程中遇到提示npmlog模块找不到的情况，可以先尝试这么做，(我没遇到过)
```shell
$ curl -0 -L https://npmjs.org/install.sh | sudo sh.
```

根据官网的建议，我们还需要再安装[watchman](https://github.com/facebook/watchman)，它的最大作用是用来监听文件状态的改变，也就是当被管理的目录下发生任何文件的增删改查它都会进行监听记录，使用它能够加速每次RN工程目录发生变化时再编译的过程，其实咱们不使用watchman也是OK的，无非就是等嘛，RN工程大了以后每次编译就出去散个步嘛。加了watchman后，针对比前后两次提交编译的请求，只编译有改动的文件，这样能够减少编译所耗费的时间。RN推荐的做法是，装，

```shell
brew install watchman
```

以上内容都完成后，此时我们RN的本地开发环境都已经弄好了。我们来创建一个工程，
```shell
$ react-native init '你的工程名字'
# 此处就以pjhubsssssss为例 #
```

此时，我们会等待一段巨特么长（长到我要骂人了）的时间，等到项目新建完了以后，我们可以看到以下的目录结构，

  <div align="center">    
  <img src="http://7xszq8.com1.z0.glb.clouddn.com/RN02.png" width = "40%" height = "40%" align=center />
  </div>

  **\__tests\__：**RN的单元测试文件夹；

  **android：**Android工程；

  **ios：**iOS工程；

  **node_modules：**RN的模块依赖管理（可以认为是Maven或pod）；

  **App.js：**相当于对应平台下的MainActivity.java或ViewController.m；

  **app.json：**RN工程的一些配置，比如工程名等；

  **index.js：**目前我对这个理解是一个bridge，Android和iOS工程分别对其读取签名。
  
  **package-lock.json和package.json：**都是npm用来管理RN的。


工程初始化后，我们此时可以如此这般运行demo

```shell
$ react-native run-ios
```
执行完命令后，会再次等到一次又特么巨长的编译时间（跟Native对比起来差距真的太明显了）。等了一会，我们会发现跑起了一个iPhone 6的模拟器。iOS工程demo就这么顺利的运行起来了，接下来，我们来好好的整一整Android。🙂

既然运行iOS工程是run-ios那么运行Android工程就一定是run-android了，看了一眼官方文档，要求我们先把模拟器运行起来。打开我一年前下的Android studio，emmmm，它开始提示我要更新一堆东西了，刚开始没管，直接去把模拟器打开，发生了各种迷之错误（心里开始怀念Xcode的各种好），比如Gradle版本、SDK位置错误等等，最后不行了，重新去下了3.0.1，所有问题全部解决。🙂

运行模拟器后执行，

```shell
$ react-native run-android
```
此时发生了一个错误，
```shell
java.lang.RuntimeException: SDK location not found. Define location with sdk.dir in the local.properties file or with an ANDROID_HOME environment variable.
```

意思就是在local.properties这个文件中没找到ANDROID_HOME，也即android SDK。解决的办法就是在Android工程目录下的local.properties文件中添加对应的android SDK路径，
```shell
sdk.dir = /Users/incloud/Library/Android/sdk
# 供参考
```

此时再执行run命令，即可看到下载了对应工程名的apk已经下载到了应用程序列表里，并且还会自动起一个终端记录对应的操作log。是的，iOS是直接帮你run起来，而Android是给你load到应用程序列表里，你自己去翻吧。2333

如果此时你并未关闭这个终端，而再次新建了一个RN工程，并且还run了它，那么会告诉你端口被占用，因为RN默认端口是8081，我们可以这么做来更改端口号，
```shell
react-native start --port 8083
```

这个问题解决后，它可能还会告诉你'...xxx has been register...'，刚开始出现这问题，我一直以为是iOS或Android工程的签名没弄好，苦恼了好久，最后没想到居然是因为上一个工程自动起的进程终端没关闭，导致下一个新建的工程运行后判断log进程终端还在，就套用了之前的终端，就导致签名冲突。

可能还会出现这么个问题，
```shell
* What went wrong:
A problem occurred configuring project ':app'.
> Failed to notify project evaluation listener.
   > com.android.build.gradle.tasks.factory.AndroidJavaCompile.setDependencyCacheDir(Ljava/io/File;)V

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

* Get more help at https://help.gradle.org

BUILD FAILED in 0s
Could not install the app on the device, read the error above for details.
Make sure you have an Android emulator running or a device connected and have
set up your Android development environment:
https://facebook.github.io/react-native/docs/android-setup.html
```

懵逼了好久，才发现原来是因为之前创建好RN工程后，打开了Android Studio，它提示有更新我就更新了，导致Android Studio和RN工程中的gradle版本不一致导致的，现在还没找到更改的手段，我只能重新init了一个工程，大家如果知道一定要告诉我🙏🙏🙏🙏

po一张完成图，
  <div align="center">    
  <img src="http://7xszq8.com1.z0.glb.clouddn.com/WechatIMG88664.jpeg
" width = "100%" height = "100%" align=center />
  </div>

---

以上就是本次React Native探路的第一步，总的来看，坑是不少，只能祝愿自己今后能避免部分坑吧。😂


[下篇：React-Native记（一）](./React-Native记（一）.md)
