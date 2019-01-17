## Flutter 三探
> 历时一个星期对 Flutter 一期调研在这篇文章中就告一段落了，这篇文章中继续完善上篇文章中利用豆瓣电影 Top250 公开 API demo。

## 前言
在这两三天的继续完善 demo 的时间中，首先是对 Flutter 在基本 UI 视觉方面的实现表示赞赏，有些地方的 UI 布局的代码编写习惯了 Flutter 的思维后，会有一个非常快速的反应。下面是具体 demo 具体的完成图：

![首页.png](https://i.loli.net/2019/01/17/5c3ff42164da7.png)

![下拉刷新.png](https://i.loli.net/2019/01/17/5c3ff4608b766.png)

![详情.png](https://i.loli.net/2019/01/17/5c3ff4ac36213.png)

整体 demo 做完后，全程都是在使用 meizu 15 这台开发机进行调试，在 flutter 的 IDE 选择上一直都在使用 `Android Studio`，在断点调试、查看渲染节点、性能对比等活动上都非常方便的解决了，依然强推！但整体没有遵循 Flutter 官方推荐的[ `BloC` ](https://cloud.tencent.com/developer/article/1345645) 设计模式，还是采用“设计模式之王”的 `MVC`，同样是考虑到了后续在对比其它跨段方案时尽可能的保证一致性。

### 数据来源
[豆瓣电影详情 API ](https://api.douban.com/v2/movie/subject/26942674)同样不需要做验证，传入对应电影的 id 即可，但会限制同一 IP 在一定间隔时间内的访问次数，如果在一定间隔时间内容访问 API 的速度过于频繁，则会拒绝服务，不过得益于 Flutter 的 `hot reload` 技术，可以不需要每次都重新拉去数据。该详情 API 多了一些更具体的数据，但依然没有达到豆瓣 App 本身那般丰富。

![《神秘巨星》电影详情数据.png](https://i.loli.net/2019/01/17/5c3ff7debdb93.png)


### 涉及 Flutter 知识点
* 下拉刷新；
* 上拉加载；
* 利用 `GestureDetector Widget` 进行页面跳转（动态路由方式）；
* 利用 `SingleChildScrollView Widget` 进行滚动视图的构建；
* 简单性能分析。

## 实践
### 目录结构
![目录结构.png](https://i.loli.net/2019/01/17/5c402f8110e56.png)

### 数据处理
电影详情 API 返回的字段更多，同样可以确认的是 Model 也一定要从网络数据源中进行抛离，这同样也为后续构建子组件回填数据时提供方便，我的电影详情 Model 如下所示：

```Dart
class MovieMember {
  String id;
  // 成员姓名
  String name;
  // 详情 URL
  String detailUrl;
  // 中清晰度头像
  String avatarUrl;

  MovieMember({
    this.name,
    this.detailUrl,
    this.avatarUrl,
  });

  MovieMember.fromJSON(Map<String, dynamic> json) {
    this.id = json['id'];
    this.avatarUrl = json['avatars']['medium'];
    this.detailUrl = json['alt'];
    this.name = json['name'];
  }
}

class MovieDetail {
  // 标题
  String title;
  // 上映年份
  String year;
  // 原名
  String originalTitle;
  // 所属国家或地区
  List<String> countries;
  // 评分
  String rating;
  // "想看"人数
  String wishCount;
  // 星星
  String stars;
  // 高清晰度海报
  String poster;
  // 电影类型
  List<String> genres;
  // 评分人数
  int ratingsCount;
  // 主要导演
  List<MovieMember> director;
  // 主要演员
  List<MovieMember> casts;
  // 简介
  String summary;

  MovieDetail({
    this.title,
    this.year,
    this.countries,
    this.rating,
    this.stars,
    this.poster,
    this.genres,
    this.ratingsCount,
    this.director,
    this.casts,
    this.summary,
    this.wishCount,
  });

  MovieDetail.fromJSON(Map<String, dynamic> json) {
    this.title = json['title'];
    this.year = json['year'];
    this.summary = json['summary'];
    this.poster = json['images']['large'];
    this.ratingsCount = json['ratings_count'];
    this.originalTitle = json['original_title'];
    this.wishCount = json['wish_count'].toString();

    this.rating = json['rating']['average'].toString();
    this.stars = json['rating']['stars'].toString();

    this.countries = new List<String>.from(json['countries']);
    this.genres = new List<String>.from(json['genres']);

    List<MovieMember> castsMembers = [];
    (json['directors'] as List).forEach((item) {
      MovieMember movieMember = MovieMember.fromJSON(item);
      castsMembers.add(movieMember);
    });
    this.director = castsMembers;

    List<MovieMember> directorMembers = [];
    (json['casts'] as List).forEach((item) {
      MovieMember movieMember = MovieMember.fromJSON(item);
      directorMembers.add(movieMember);
    });
    this.casts = directorMembers;
  }
}
```

在写 `MovieDetail` Model 时发现电影详情 API 返回数据源中的“演员”数据存在多字段必要数据，为了后续方便调用同样也抽离了一个 `MovieMember` Model（二期调研估计会继续做演员详情）。

![演员数据格式.png](https://i.loli.net/2019/01/17/5c40188d4b459.png)

网络数据的获取因为有了上篇文章的铺垫，这次再写一个速度明显快了很多，一期调研完整的网络层方法如下：
```Dart
import 'dart:io';
import 'dart:convert';
import 'package:movie_top_250/Model/movieModel.dart';

class MovieAPI {
  Future<Movies> getMovieList(int start) async {
    var client = HttpClient();
    int page = start * 50;
    var request = await client.getUrl(Uri.parse(
        'https://api.douban.com/v2/movie/top250?start=$page&count=50'));
    var response = await request.close();
    var responseBody = await response.transform(utf8.decoder).join();
    Map data = json.decode(responseBody);
    return Movies.fromJSON(data);
  }

  Future<MovieDetail> getMovieDetail(String movieId) async {
    var client = HttpClient();
    var request = await client.getUrl(Uri.parse(
        'https://api.douban.com/v2/movie/subject/$movieId'));
    var response = await request.close();
    var responseBody = await response.transform(utf8.decoder).join();
    Map data = json.decode(responseBody);
    return MovieDetail.fromJSON(data);
  }
}
```

### 下拉刷新与上拉加载
下拉刷新的整体与 Native 开发思路一致。上拉加载有根据业务有很多种实现方案，因为只是为了做验证性 demo ，直接采取了使用“静默加载”的思路，当然因为一次加载数据量太大（一页 50 条），所以在快速滑动列表时会导致下一页数据未载入等待的体验。如果不考虑过多的定制化操作，直接使用 flutter 系统组件是一件非常舒服的事情。完整代码如下：

```Dart
import 'package:flutter/material.dart';
import 'package:movie_top_250/Service/movieApi.dart';
import 'package:movie_top_250/Model/movieModel.dart';
import 'package:movie_top_250/View/List/movieListViewRowWidget.dart';

class MovieWidget extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return _DouBanMovieState();
  }
}

class _DouBanMovieState extends State<MovieWidget> {
  // 数据源
  List<Movie> movies = [];
  // 分页
  int page = 0;

  @override
  void initState() {
    super.initState();
    _requestData();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('豆瓣电影 Top250'),
      ),
      body: new RefreshIndicator(
        child: _buildList(context),
        onRefresh: _requestData,
        color: Colors.black,
      ),
    );
  }

  // 下拉刷新
  Future<Null> _requestData() async {
    movies.clear();
    await MovieAPI().getMovieList(0).then((moviesData) {
      setState(() {
        movies = moviesData.movies;
      });
    });
    return;
  }

  // 上拉加载
  _requestMoreData(int page) {
    print('page = $page');
    MovieAPI().getMovieList(page).then((moviesData) {
      setState(() {
        movies += moviesData.movies;
      });
    });
  }

  // body List Widget
  Widget _buildList(BuildContext context) {
    var screenWidth = MediaQuery.of(context).size.width;
    if (movies.length != 0) {
      return ListView.separated(
          itemBuilder: (context, index) {
            // 还剩 15 条数据的时去拉取新数据
            if (movies.length - index == 15) {
              _requestMoreData(++page);
            }
            return new Container(
              width: screenWidth,
              child: buildListRow(index, movies[index], context),
            );
          },
          separatorBuilder: (context, index) => Divider(
                height: 1,
              ),
          itemCount: movies.length);
    } else {
      return Center(
        child: CircularProgressIndicator(),
      );
    }
  }
}
```

### UI 分析
![豆瓣电影详情](https://i.loli.net/2019/01/17/5c40165537e3d.jpg)

上文中也已经说到，因为豆瓣电影详情公开 API 所暴露出的数据有限，导致未能 100% 的重写。经过分析后主要将页面分为了以下几部分：

![豆瓣电影详情 UI 布局分析](https://i.loli.net/2019/01/17/5c401710e02cd.jpg)

#### 第一部分
第一部分与上篇文章中所讲述的布局编写思路大部分一致，对于我自己来说有个需要注意的地方，在第一部分中有个“豆瓣电影排名”的 `badge`，原本打算是用 `RichText Widget` 进行实现的，但翻完属性后发现并没有提供 `decoration` 字段进行修饰，最后直接使用了两个 `DecoratedBox Widget` 作为父容器，在其 `decoration` 属性下使用 `BoxDecoration Widget` 完成“一左一右”圆角的 `badge` 组件编写，flutter 在组件“半圆角”的实现过程比 iOS 原生实现的代码量上少太多了（不封装的话），实现代码如下：

```Dart
Widget _buildBadge(int index, MovieDetail movieDetail) {
  index++;
  return new Row(
    children: <Widget>[
      DecoratedBox(
          child: Padding(
              padding: EdgeInsets.fromLTRB(7, 3, 7, 3),
              child: Text('No.$index',
                  style: TextStyle(color: Colors.brown, fontSize: 14))),
          decoration: new BoxDecoration(
              color: Colors.orangeAccent,
              borderRadius: BorderRadius.only(
                  topLeft: Radius.circular(5),
                  bottomLeft: Radius.circular(5)))),
      DecoratedBox(
          child: Padding(
              padding: EdgeInsets.fromLTRB(7, 3, 7, 3),
              child: Text('豆瓣Top250',
                  style: TextStyle(color: Colors.orangeAccent, fontSize: 12))),
          decoration: new BoxDecoration(
              color: Colors.black45,
              borderRadius: BorderRadius.only(
                  topRight: Radius.circular(5),
                  bottomRight: Radius.circular(5)))),
    ],
  );
}
```

在实现“想看”和“看过”两个按钮组件时，我使用了 `RaisedButton Widget` 。一开始是这么写的：

```Dart
new RaisedButton(
  onPressed: null,
  color: Colors.white,
  child: new Text('B'),
  textColor: Colors.black,
)
```

当显示出来后，不管怎么调整样式、修改颜色、文字等都不管用。最后带着郁闷的心情浏览官方文档，居然发现了这么一段话：

![RaisedButton 官方解释](https://i.loli.net/2019/01/17/5c401d92c1bdf.png)

嗯，就算我们并不想让这个 Button 响应任何点击事件也不能给这个属性置空，并且也不能删除，因为这是个必须参数......这点需要注意。让同样也没想到的是 `RaisedButton` 没有 `text` 或类似设置按钮文本的属性，而是给了一个 `child` 属性，被 `UIButton` 虐过几次后在 `Flutter` 中看到某个组件提供了 `child` 属性现在就两眼放光！完成的代码如下：

```Dart
Widget _buildButton() {
  return new Padding(
    padding: EdgeInsets.fromLTRB(0, 20, 0, 0),
    child: new Row(
      children: <Widget>[
        new Padding(
          padding: EdgeInsets.fromLTRB(0, 0, 10, 0),
          child: new RaisedButton(
            onPressed: () {},
            color: Colors.white,
            child: new Row(
              children: <Widget>[
                new Padding(
                  padding: EdgeInsets.fromLTRB(0, 0, 5, 0),
                  child: new Icon(
                    Icons.remove_red_eye,
                    size: 18,
                    color: Colors.orange,
                  )
                ),
                new Text('想看',
                  style: new TextStyle(
                    color: Color.fromRGBO(100, 100, 100, 1),
                    fontSize: 16,
                    fontWeight: FontWeight.w700,
                  ),
                ),
              ],
            ),
            textColor: Colors.black,
          ),
        ),
        new RaisedButton(
          onPressed: () {},
          color: Colors.white,
          child: new Row(
            children: <Widget>[
              new Padding(
                  padding: EdgeInsets.fromLTRB(0, 0, 5, 0),
                  child: new Icon(
                    Icons.star_border,
                    size: 18,
                    color: Colors.orange,
                  )
              ),
              new Text('看过',
                style: new TextStyle(
                  color: Color.fromRGBO(100, 100, 100, 1),
                  fontSize: 16,
                  fontWeight: FontWeight.w700,
                ),
              ),
            ],
          ),
          textColor: Colors.black,
        ),
      ],
    ),
  );
}
```

#### 第二部分
这部分布局与上篇文章所讲述的内容都是一致的。并且我也偷懒了，主要是没有太多值得花费心思的地方,都是常规的布局.本来想实现下“进度条”，但无奈并没有真实数据，也就懒得弄了。完整代码如下：

```Dart
import 'package:flutter/material.dart';
import 'package:movie_top_250/Model/movieModel.dart';


Widget movieDetailStarWidget(MovieDetail movieDetail) {
  return new DecoratedBox(
    decoration: new BoxDecoration(
        color: Color.fromRGBO(65, 46, 37, 1),
        borderRadius: BorderRadius.all(Radius.circular(5))),
      child: new Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          new Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: <Widget>[
              new Padding(
                  padding: EdgeInsets.all(10),
                  child: new Text(
                    '豆瓣评分™',
                    style: TextStyle(color: Colors.white),
                  )
              )
            ],
          ),
          _buildRatingStar(movieDetail),
        ],
      )
  );
}

Widget _buildRatingStar(MovieDetail movieDetail) {
  List<Widget> icons = [];
  int fS = int.parse(movieDetail.stars) ~/ 10;
  int f = 0;

  while (f < fS) {
    icons.add(new Icon(Icons.star, color: Colors.orange, size: 15));
    f++;
  }
  while (icons.length != 5) {
    icons.add(new Icon(Icons.star,
        color: Color.fromRGBO(220, 220, 220, 1), size: 15));
  }

  return new Padding(
    padding: EdgeInsets.fromLTRB(0, 5, 0, 10),
    child: new Column(children: <Widget>[
      new Padding(
        padding: EdgeInsets.fromLTRB(5, 0, 0, 10),
        child: new Text(
          movieDetail.rating,
          style: new TextStyle(
            fontSize: 35,
            fontWeight: FontWeight.w500,
            color: Color.fromRGBO(220, 220, 220, 1),
          ),
        ),
      ),
      new Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: icons
      ),
    ]),
  );
}
```


#### 第三部分
这部分涉及到了长文本，flutter 中同样也没有提供长文本显示组件，但好在 flutter 的 `Text Widget` 本身就适用于长文本展示的组件，默认开启 `softWrap` 属性（自动换行）。`Text Widget` 会从自身节点树里一直向上寻找能够提供宽度约束的父组件，并以此作为单行文本最长显示宽度，这点还是比较惊讶的，省了非常多的事情。完整的代码如下：

```Dart
import 'package:flutter/material.dart';
import 'package:movie_top_250/Model/movieModel.dart';

Widget movieDetailSummaryWidget(MovieDetail movieDetail) {
  return new Padding(
    padding: EdgeInsets.all(15),
    child: new Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: <Widget>[
        new Padding(
          padding: EdgeInsets.fromLTRB(0, 0, 0, 10),
          child: new Text(
            '简介',
            style: new TextStyle(
              fontSize: 20,
              color: Colors.white,
              fontWeight: FontWeight.w600,
            ),
          )
        ),
        new Text(
          movieDetail.summary,
          style: new TextStyle(
              color: Colors.white,
              fontSize: 15,
          )
        )
      ],
    )
  );
}
```

当数据展示出来后，发现超出当前页面的显示范围了，这也是意料之中。按照之前的做法，会把 `UIScrollView` 作为当前页面所有原始的父容器，等所有 UI 元素都回填数据重新渲染完后，再把位于最底部的 UI 组件 bottom 值赋给 `scrollView.contentSize`。

翻了 flutter 文档后，发现了提供常规滑动视图能力的 Widget 不只一个，最后选择了 `SingleChildScrollView`。本以为设置滑动区域的步骤也会向在 Native 中那般麻烦，但实际上只需要把需要滑动视图组件的父节点赋给 `child` 属性即可，`SingleChildScrollView` 同样会自动的拓展自己的滑动区域进行适配，如下所示：

```Dart
Widget _buildBody(BuildContext context) {
    // 数据源没来时展示 loading
    if (movieDetail == null) {
      return new Center(
        child: new CircularProgressIndicator(),
      );
    } else {
      return new SingleChildScrollView(
        child: new Padding(
            padding: EdgeInsets.all(10),
            child: new Column(children: <Widget>[
              movieDetailHeaderWidget(rankIndex, movieDetail, context),
              movieDetailStarWidget(movieDetail),
              movieDetailSummaryWidget(movieDetail),
              movieDetailMemberWidget(movieDetail),
            ])
        ),
      );
    }
}
```

#### 第四部分
这部分是整体比较纠结的地方，到底是基于 `GridView Widget` 还是 `SingleChildScrollView Widget` 配合着其它布局 Widget 去做呢？如果这在 iOS 中，我会毫不犹豫的选择 `UICollectionView` 进行构建，因为又快又好～

最后还是抱着“又快又好”目的出发，选择了 `SingleChildScrollView Widget` 配合着其它布局 Widget 去做。需要注意是的 `SingleChildScrollView Widget` 默认是纵向滚动，该部分豆瓣 App 进行的横行滚动，需要改变滚动视图方式。完整代码如下：

```Dart
import 'package:flutter/material.dart';
import 'package:movie_top_250/Model/movieModel.dart';

Widget movieDetailMemberWidget(MovieDetail movieDetail) {
  List<Widget> memberWidgets = [];

  for (MovieMember member in movieDetail.director) {
    memberWidgets.add(_buildMemberWidget(member, true));
  }

  for (MovieMember member in movieDetail.casts) {
    memberWidgets.add(_buildMemberWidget(member, false));
  }

  return new SingleChildScrollView(
    scrollDirection: Axis.horizontal,
    child: new Row(
      children: memberWidgets,
    ),
  );
}

Widget _buildMemberWidget(MovieMember member, bool isDirector) {
  var col = new Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
      new Container(
        width: 110,
        height: 160,
        decoration: new BoxDecoration(
          image: DecorationImage(image: NetworkImage(member.avatarUrl)),
          borderRadius: new BorderRadius.all(
            const Radius.circular(8.0),
          ),
        ),
      ),
      new Text(member.name,
        style: new TextStyle(
            color: Colors.white,
        ),
      ),
    ],
  );


  if (isDirector) {
    col.children.add(
      new Text(
        '导演',
        style: new TextStyle(
            fontSize: 13,
            color: Color.fromRGBO(150, 150, 150, 1)
        ),
      )
    );
  } else {
    col.children.add(
        new Text(
          '演员',
          style: new TextStyle(
              fontSize: 13,
              color: Color.fromRGBO(150, 150, 150, 1)
          ),
        )
    );
  }

  return new Container(
    width: 110,
    child: new Padding(
      padding: EdgeInsets.all(15),
      child: col,
    ),
  );
}
```

## 分析工具
在本 demo 中的 `ListView Widget` 未做太多优化的地方，所以会导致在启动 App 完成后直接开始上拉页面会体验到卡顿，但实际上的做法是先把当前页数据源的条数给 `ListView` 设置上，当用户停止滑动页面后，再开始 load 当前在可视区域范围内的 `ListViewRow Widget` 相关子组件数据。这一点优化在 iOS 上通过 `UITableView` 配合 `RunLoop` 就可以解决，但在对 flutter 的一期调研中并未打算开始此项调优工作。

使用各种跨平台工具最令人感到窒息的莫过于调试了，基于 `JSCore` 比如 `Weex`、`React-Native` 等框架还行，能够利用 web 开发者工具搭配进行。但比如 `Xamarin`、`Qt` 等框架想要进行调试基本上就比较费劲了，如果框架开发者不提供一些功能完备的 IDE 或插件，调试几乎等于噩梦。好在 `Android Stuido` 对 flutter 的支持是相当丰富，具体见下图：

![查看当前页面节点树](https://i.loli.net/2019/01/17/5c402c1916281.png)

![分析工具](https://i.loli.net/2019/01/17/5c4030aedce1e.png)

![分析工具.png](https://i.loli.net/2019/01/17/5c4031b790f79.png)

![性能相关（部分）](https://i.loli.net/2019/01/17/5c402f3580f77.png)

## 其它
### webView
今天原本还想做跳转 `webView`，以为也只是直接调个 `webView Widget`，填入 `requestUrl` 属性就完事了。但当我输入 web ，IDE 并未提示任何相关信息时，开始发觉有点不太对劲，不会 flutter 没有提供 `webView Widget` 吧？仔细浏览后，确认 flutter 还真的没有提供官方 `webView` 组件，但在 Pub 上已经有了对应的插件。

接着去掘金的 flutter 交流群里咨询，讨论在 flutter 中 `webView` 以及 `JSBridge` 最佳思路，最后讨论出了两个插件：

* webView 插件（带 JSBridge）：https://pub.flutter-io.cn/packages/interactive_webview
* webView 插件：https://pub.dartlang.org/packages/flutter_webview_plugin

对于 `webView` 这块不是特别满意，而且看了 flutter 在 github 上的 issue，推荐自己做一个 `webView plugin`，暴露给 flutter 进行调用，这样可以最大程度上的降低基础组件重写成本。仔细一想，其实还是不满意，所以这部分内容也延后到二期调研中了。

### `StatefulWidget` 和 `StatelessWidget` 的选择
对于开发一个新的组件时，到底是基于 `StatefulWidget` 还是 `StatelessWidget` ，我认为只需要明确两个概念即可：

* 是否需要更新组件数据源；
* 是否需要利用组件各种生命周期；

如果以上两个条件都符合，那就选择 `StatefulWidget`。

## 总结
### 源码地址
本次 flutter 一期调研学习产出的豆瓣电影 Top250 demo 链接地址：[movie_top_250](https://github.com/windstormeye/flutter-practices/tree/master/movie_top_250)

flutter 的这种“声明式”编码体验，我认为对于第一次接触的新手来说，肯定有需要一定的学习成本，当逼迫自己去熟悉开发思路后，就会觉得真的很过瘾。在一期调研的学习中，没有涉及到的方面有：

* 设计模式；
* `webView` 及 `JSBridge`；
* flutter 与 native 的交互；
* 复杂 UI 的构建；
* 音视频处理（这还是得通过 native 进行暴露）；

以上几块是构建一个 App 所需要具备的基本组件。经过一期学习后，对 flutter 也有了自己的理解，给我最大的感受是因为不用像 `React-Native`、`Weex` 等需要回调节点树给中间层通知 native 利用原生 UI 框架进行渲染，首先在 UI 绘制速度上已经远超其它框架，这点是毋庸置疑的。但因为 flutter 还太年轻，一些基础设施和社区都做得不算太好。所以如果非要选择一个跨端技术投入实际开发中，我还是会选择 `React-Native`，所以将重新捡起来 `React-Native`， 对其同样重新进行一期调研。

