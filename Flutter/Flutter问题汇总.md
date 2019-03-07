# Flutter 问题汇总
## 环境配置
根据[ `flutter` 中文官网](https://flutterchina.club)上所引导的步骤进行配置，中途可以根据 `flutter doctor` 命令进行检查相关依赖是否配置完成。

### 设备
* iOS: iPhone 7, iOS 12.1.2
* Android: meizu 15, Andriod 7.1.1

### 遇到的问题
* 在环境配置中，官方推荐使用 `Andriod Studio` 进行开发，因为体验是最好的，当然同时也支持 `VS Code` 和 `IntelliJ`。因为开发机“常年”连接公司内网，导致无法在 `Andriod Studio` 中下载 `Dart` 和 `Flutter` 插件，尝试好几次，网上的资料都翻遍了，突然灵光一闪！我特么这是在内网啊！切回外网后，一切顺畅......

## 初体验
Flutter 官方上说的优势之一为“热重载”，新建 flutter 测试项目分别运行在 iOS 和 Andriod 两台测试设备上，iOS 的热重载只要每次 `cmd + s` 即可，但 Andriod 需要执行两次，看第一次打印出来的信息提示已经完成 `hot reload`，但设备上什么都没出现，必须执行第二次 `cmd + s` 操作后，才能看到真正的 `hot reload` 的效果。

![左：iPhone 7，右：meizu 15](https://i.loli.net/2019/01/10/5c36f04c618e7.jpg)

flutter 官网上对于“热重载”是这么描述的：

> 通过将更新后的源代码文件注入正在运行的 `Dart` 虚拟机（VM）中来实现热重载。在虚拟机使用新的的字段和函数更新类后，`Flutter` 框架会自动重新构建 `widget` 树，以便您快速查看更改的效果。

所以对于在 meizu 15 上需要执行两次保存操作才能触发“热重载”后的效果展示，我的推测是，在第一次执行保存操作时要么没有把新更新后的代码注入进 `Dart` 虚拟机中，要么就是注入了但未触发重新自动构建 `widget` 树。

### 渲染
![Flutter 在 iOS 上的视图层级](https://i.loli.net/2019/01/10/5c37187ca736f.png)

### 差异点
* 入口的 Main 函数入口使用了 `=>` 语法糖，官方说是“这是 `Dart` 中单行函数或方法的简写”：

```Dart
void main() => runApp(new MyApp());

// 我的推测：上下两者相等，论简洁性，确实是好看一丢丢
void main() { runApp(new MyApp()) }
```

* 每一个 `Widget` 都会有一个 `build()` 方法，用于描述如何根据其他较低级别的 `widget` 来显示自己。我的理解就是 `initView` 方法；

* 在 `Dart` 中“万物”（包括布局）都是 `Widget`，这点就类似与 `Objective-C` 中的“万物”都是 `NSObject`；
* `Scaffold Widget` 是 `Material library` 中的一个 `Widget`，提供了 `Material` 风格的基本组件。
* Flutter 中并没有类似 iOS 中的 `UITableViewCell`，直接在 `ListView Widget` 中构建了 `cell`，正是因为没有 `cell` 的概念，所以原本每个 `cell` 之间的“分割线”也需要手动使用 `Divider Widget` 进行索引模拟。推荐一篇关于 `Scaffold Widget` 的[内容介绍](http://flutter.link/2018/03/20/Scaffold/)

* Flutter 的 `Widget` 分为 `StatefulWidget（有状态）` 和 `StatelessWidget（无状态）` 两种，这跟在 iOS 中只要是继承了 `UIResponder` 就具备与用户产生交互进行状态的改变不一样。在 flutter 中如果我们需要实现设计要这个组件是否需要有状态的改变。

### 一些简单操作
#### 当打开一个工程时
`flutter packages get` 来下载工程中所依赖的库。

#### 格式化代码
`Dart` 疯狂嵌套的代码风格已经被吐槽烂了，好在可以在写完代码后，利用 `Android Studio` 中提供的 `Dart` 格式化代码工具：选择任何一个 `Dart` 代码文件，右键选择“Reformat Code with dartfmt”，代码格式立马变得好看了许多。

### 总结
经过这次对 Flutter 的初体验，对其惊叹的地方有：
* 真的做到了一套代码可以“无脑”运行在 iOS 和 Android 两个平台上，使用 `Andriod Studio` 编写完主体代码后，完全不需要做任何平台差异化设置，直接选择不同平台设备直接运行即可，在加上真的脱离了 `JS Core` 的“热重载”技术，在 iOS 上的开发体验非常流畅和方便！
* 在 iOS 上真的抛弃了 `UIKit` 的所有内容，全都基于 `Skia` 自己渲染，这点跟 `Texture` 在 UI 渲染上有异曲同工之处。
* `Dart` 这门语言本身有着与 `JSX` 类似的代码风格痕迹，尤其是对 `Widget` 做属性的定义时，但从整体上来看因为前身是准备要替代 `JS`，所以在很多地方也有 `JS` 痕迹，在一些细节上又透露着 `Java` 的微小细节，所以从语言本身的上手难度不算大，并没有在语法层面上做出太多的革新。
* 强烈推荐使用 `Android Studio` 进行开发！！！
* 创建 Flutter 工程下的 iOS 平台工程居然主体基于 `Swift`，这点让我十分意外！

目前来看不满意的地方只有一个：
* 在 iOS 上的长列表滑动卡顿十分严重！！！在快速滑动下，估计只有两三帧，而且每一个 `ListTitle Widget` 上只放了一个 `Text Widget` 啊！太辣眼睛了......[视频在此](https://www.bilibili.com/video/av40402669/)


## 二探 Flutter
### 一些细节
* `MaterialApp` 下的 `title` 属性代表的是在 Android 任务管理器中的名称（iOS 下只看 app name），`home` 属性下的 `appBar` 中所返回的 `AppBar` 中 `title` 属性才是定义 app 当前页面的标题；

### http
想要在 `Flutter` 中使用 `http` 请求，需要先在 `pubspec.yaml` 文件中加入对应依赖：
```yaml
dependencies:
  # ...
  http: ^0.12.0+1
```

随后可在对应 `dart` 文件中进行引入：
```dart
import 'package:http/http.dart' as http;
```

## HTTP 相关
### 刷新数据
因为 Flutter 对数据和视图已经进行了绑定，如果想要在网络请求完成后刷新视图所绑定的数据源，需要使用 `setState` 方法进行数据源状态的刷新。当然，使用 `setState` 方法的前提得是我们的 `Widget` 得是一个 `StatefulWidget` ，具备“状态改变的能力”。

### 异步加载后获取数据
通过 flutter 的自带 HTTP 请求库开启异步获取数据后，要求把数据为 `Future` 类型格式，以便在组件中进行“懒加载”，对于 `Future` 类型数据的解析可以采用如下方法（）：
```Dart
class MovieAPI {
  Future<Movies> getMovieList(int start) async {
    var client = HttpClient();
    var request = await client.getUrl(Uri.parse(
        'https://api.douban.com/v2/movie/top250?start=$start&count=100'));
    var response = await request.close();
    var responseBody = await response.transform(utf8.decoder).join();
    Map data = json.decode(responseBody);
    return Movies.fromJSON(data);
  }
}

// 上拉加载
_requestMoreData(int page) {
  MovieAPI().getMovieList(page).then((moviesData) {
    setState(() {
      movies = moviesData.movies;
    });
  });
}
```

### 点击事件
在 iOS 和 Android 中所有的 `View` 都可以添加点击或其它多手势事件，但在 Flutter 中除了少数几个自带 `onPress` 或 `onTap` 事件的 `Widget` ，剩下绝大部分 `Widget` 都不带事件，需要我们自己使用 `GestureDetector Widget` 作为父组件进行包裹。

```Dart
return GestureDetector(
  onTap: () {
    // 写下单机后触发的内容，当然还有双击、长按等事件
  },
  child: yourWidget,
);
```

### Flutter Widget Inspector

### 关于 `RaisedButton` 问题
```Dart
new RaisedButton(
  onPressed: null,
  color: Colors.white,
  child: new Text('B'),
  textColor: Colors.black,
)
```

如果不对 `onPressed` 设置处理事件，则对 `RaisedButton` 设置的所有修饰都不生效。
