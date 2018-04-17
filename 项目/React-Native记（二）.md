---
title: React-Native记（二）
date: 2018-02-14 11:09:50
tags:
- React-Native
---

停了一段时间后RN的学习又持续了💪。在停止的这段时间中主要是去做了关于iOS的一些细节加强，这部分内容也是自己之前一直想去总结出来容易遗忘和出错的地方。

随后跟进的RN学习主要是学习`ScrollView相关使用`、`手撸轮播图`、`ListView的使用`、`tabBar的使用`、`Navigator的使用`、`网络请求的简单认识`，也快过年啦，先在此祝各位看友过年好~，因为回到老家网实在是不好，很多事情不方便去做😔，一些零零散散的琐事也时不时的打扰着自己，只能每天推进一点点。

在这段学习RN的时间中，我主要认识到了RN“真的很垃圾”🙂（注意：这是个双性词！），做一部分内容比Native真的方便超多的（甚至都觉得有些傻），至于哪些地方方便，之前文章中也说的七七八八了，但是重点在于做一些比如轮播图、TabBar这些Native支持得非常好的组件，在RN中就尴尬得不行（后文细谈）。所以在这段时间的学习中我差点萌生了“劝退自己”的想法，因为实在是太恶心了，先不管RN真正的精髓与我目前所理解的到底对不对得上，单是这几个基本组件在RN中实现起来都如此“恶心”（用官方的话来说，底层就是这么做的🙄），说句良心话，我实在是受不了。而且RN目前来说，虽然说是“Learn once，run anywhere”（有可能记错了？），但是iOS和Android现在位于RN生态中的分割还是太大，也有可能是自己的功夫不到家，单是一个tabBarIOS我就受不了（当然，也有可能是因为RN还需要再发展？）

我们先来看看使用ScrollView实现的轮播图效果

<img src="https://i.loli.net/2018/02/14/5a843993733f4.gif" width = "40%" height = "40%" align=center />


看上去效果还行，能够实现`触摸拖动中断`，但是因为整体没搭建完，并未能实现滑动整个View让轮播图进行中断停止（具体效果可参考京东）。整体实现思想跟iOS Native是一样的，同样都是需要ScrollView作为轮播容器，使用Image标签作为轮播实体，其与原生ScrollView使用起来无二意，就是基于iOS的runtime的封装
* **需要设置高度**
    * 直接设置高度（不推荐）
    * 在CSS中不加flex：1。让ScrollView中的内容自动的填充扩张

* ScrollView内部的其他响应者尚无法阻止ScrollView本身成为响应者，ScrollView在其所有子控件的最上层，所以ScrollView内部的其它响应者并不能阻止ScrollView本身成为响应者

```js
<ScrollView
    ref={'scrollView'}
    horizontal={true}
    pagingEnabled={true}
    showsHorizontalScrollIndicator={false}
    onMomentumScrollEnd={(e) => this.onAnimationEnd(e)}
    onScrollBeginDrag={(e) => this.onScrollBeginDrag(e)}
    onScrollEndDrag={(e) => this.onScrollEndDrag(e)}
    >
    {this.renderChildView()}
    </ScrollView>

    renderChildView() {
      var childViews = [];
      var imgArr = imageData.data;
      for (var i = 0; i < imgArr.length; i++) {
          var imgItem = imgArr[i];
          console.log(imgItem)
          childViews.push(
              <Image key={i} source={require('./img/lunbo1.png')} style={{width: screenWidth, height: 120}}/>
          );
      }
      return childViews;
  }
```

以上就是轮播图的架构（非常简单）。数据结构如下，用的是本地图片资源，（这里有个大坑，后文详解）

```json
{
  "data" : [
    {
      "img" : "lunbo1",
      "title" : "2333"
    },
    {
      "img" : "lunbo2",
      "title" : "2333"
    },
    {
      "img" : "lunbo3",
      "title" : "2333"
    }
  ]
}
```

想要让轮播图在一定时间间隔后进行自动轮播，必不可少的需要一个定时器进行时间的计算，据部分资料显示，ES5需要一个`react-timer-mixin`组件，而采用ES6后可以大大减少累赘的写法，
```js
 // 组件都载入完毕后，调起计时器方法
  componentDidMount() {
    this.startTimer();
  }

  startTimer() {
    var scrollView = this.refs.scrollView;
    var imgCount = imageData.data.length;
    // 有setTimeOut和setInterval，需注意分清
    this.timer1 = setInterval(() => {
        var activePage = 0;
        // 判断当前分页
        if ((this.state.currentPage + 1) >= imgCount) {
            activePage = 0
        } else {
            activePage = this.state.currentPage + 1;
        }

        this.setState({
            currentPage: activePage
        });

        // 进行偏移
        var offSetX = activePage * screenWidth;
        scrollView.scrollResponderScrollTo({x: offSetX , y: 0, animated: true})}, 1000);
  }

  // 开始拖拽
  // 需要把ScrollView自身作为参数传入
  onScrollBeginDrag(e) {
      // ES6清除计时器
      clearInterval(this.timer1);
  }

  // 结束拖拽
  onScrollEndDrag(e) {
      this.startTimer();
  }
```

此时运行起工程，就能够发现轮播图的效果已经实现了，但是如果我们想要添加一个分页指示器那该怎么做呢？当时我的脑子就冒出了RN肯定也是封装了UIPageController，但又被事实啪啪啪的打脸，根本没有，查阅了一番资料后发现居然有网友通过了HTML中的特殊转义字符`&bull;`达到了效果。

这是补充完的render方法，
```js
render() {
    return (
    <View style={styles.container}>            <ScrollView
            ref={'scrollView'}
            horizontal={true}
            pagingEnabled={true}
            showsHorizontalScrollIndicator={false}
            onMomentumScrollEnd={(e) => this.onAnimationEnd(e)}
            onScrollBeginDrag={(e) => this.onScrollBeginDrag(e)}
            onScrollEndDrag={(e) => this.onScrollEndDrag(e)}
        >
            {this.renderChildView()}
        </ScrollView>
        {/* 返回指示器 */}
        <View style={styles.pageViewStyle}>
            {this.renderPageCircle()}
        </View>
    </View>
    );
  }
```

这是补充完的`renderChildView`和`renderPageCircle`方法，

```js
// 轮播图创建方法
renderChildView() {
    var childViews = [];
    var imgArr = imageData.data;
    for (var i = 0; i < imgArr.length; i++) {
        var imgItem = imgArr[i];
        console.log(imgItem)
        childViews.push(
            <Image key={i} source={require('./img/lunbo1.png')} style={{width: screenWidth, height: 120}}/>
        );
    }
    return childViews;
  }

// 分页指示器创建方法
renderPageCircle() {
    var indicatorArr = [];
    var style;
    var imgArr = imageData.data;
    for (var i = 0; i < imgArr.length; i++) {
        style = (i === this.state.currentPage) ? {color: 'orange'} : {color: '#ffffff'};
        indicatorArr.push(
            <Text key={i} style={[{fontSize: 25}, style]}>&bull;</Text>
        )
    }
    return indicatorArr;
}
```

以上就是使用RN实现的简单轮播图组件demo，整体来看比iOS Native实现起来更为方便快捷（重点是简明🙂，项目工程地址文后），同样也是通过获取ScrollView的偏移量进行操作，从思想上看给人感觉全都是一样的呢🙂，接着我们来看看RN中的ListView。

在学习ListView过程中实现了三个小demo，普通列表展示、吸顶列表展示（对比iOS tableView的group样式）、九宫格。其中，给我的感觉如果只是使用到ListView做一些简单的数据展示，那真的是无比的舒服，并且得益于RN自身的架构设计，我们能够很好的处理数据源的切换。

使用ListView需要这两步：
* 创建一个ListView.DataSource数据源，给它传递一个普通的数据数组；

* 使用数据源(data source)实例化一个ListView组件,定义一个回调函数，函数接受数组中的每个数据作为参数，并返回一个可渲染的组件(就是每一个Cell)

普通列表的实现方法在吸顶列表中有重复，在此我们只对吸顶列列表做讲解（在讲解中大家将会看见是有多么的恶心🙂）

首先我们需要给ListView设置数据源，而ListView的数据源设置带了我一个无比的震撼（太尼玛麻烦了🙂）
```js
  constructor(props) {
    super(props);
    this.state = {
      dataSource: new ListView.DataSource({
        rowHasChanged: (r1, r2) => r1 !== r2
      })
    };
  }
```

通过以上代码我们就定义好了关于ListView的数据加载方法，当r1行不等于r2行时，才进行ListView的数据加载。我们先不管中间数据源如何更替，我们先来看如何填充ListView的核心数据源，

```js
this.setState({
      dataSource: this.state.dataSource.cloneWithRowsAndSections(dataBlob, sectionIDs, rowIDs)
    });
```

通过以上方法，我们会调起RN的setState异步赋值方法，其实我觉得RN从架构上就是MVVM（不知道猜的对不对），当我们在setState方法中更改了相关的属性值，使用到这些相关属性值的对应组件会自动异步的刷新UI，这点超强大的好不好🙂。在iOS中单是要实现MVVM架构思想，就得拉出KVO这种看上去非常难受的东西🙂。

在ListView中要实现黏性头部，需要使用 cloneWithRowsAndSections方法，将 dataBlob(object), sectionIDs (array), rowIDs (array) 三个值传进去。

如果你曾经有过iOS开发经验（没有也行），要实现的吸顶效果，实际上就是对ListView做了分组，取名为Section，组内的每一行称之为Row，因此我们实现section和row的数据源赋值，这些东西都需要在页面初始化方法中得到解决，因此补充完的constructor方法为，

```js
constructor(props){
    super(props);
    // 初始化section方法
    var getSectionData = (dataBlob, sectionID) => {
      return dataBlob[sectionID];
    }

    // 初始化row方法
    var getRowData = (dataBlob, sectionID, rowID) => {
        return dataBlob[sectionID + ":" + rowID];
    }

    this.state = {
        dataSource: new ListView.DataSource({
        // 绑定section数据刷新
        getSectionData: getSectionData,
        // 绑定row数据刷新，只更新渲染数据变化的那一行  ，rowHasChanged方法会告诉ListView组件是否需要重新渲染当前那一行。
        getRowData: getRowData,
            rowHasChanged: (r1, r2) => r1 !== r2,
            sectionHeaderHasChanged: (s1, s2) => s1 !== s2,
        })
    };
};
```

我们需要搭建的界面代码如下所示，
```js
render() {
    return (
      <View style={styles.containerStyle}>
        // 先假装有一个NavigationBar🙂
        <View style={styles.headerViewStyle}>
          <Text style={{color: "white", fontSize: 25}}>PJHubs</Text>
        </View>
        <ListView
          dataSource={this.state.dataSource}
          // ListViewRow的数据绑定
          renderRow={this.renderRow}
          // ListViewSection的数据绑定
          renderSectionHeader={this.renderSectionHeader}
          // 设置Cell内部样式
          contentContainerStyle={styles.listViewStyle}
        />
      </View>
    )
  }

  // 创建ListViewRow
  renderRow(rowData) {
      return (
        <TouchableOpacity activeOpacity={0.5}>
          <View style={styles.innerViewStyle}>
            <Image source={require("./wine.png")} style={styles.leftImageStyle} />
            <Text style={{fontSize: 17}}>{rowData.name}</Text>
          </View>
        </TouchableOpacity>
      );
  }
  // 创建ListViewSection
  renderSectionHeader(sectionData, sectionID) {
    return(
      <View style={styles.sectionViewStyle}>
        <Text style={{marginLeft: 10,}}>{sectionData}</Text>
      </View>
    )
  }
```

因为ListView的数据加载属于耗时操作（不管数据在本地还是网络上），RN官方推荐所有的耗时操作统统给我放到`componentDidMount`方法中（组件加载完毕后），因此补充完的`componentDidMount`方法如下所示，

```js
componentDidMount() {
    this.loadDataFromJson()
  }

  loadDataFromJson() {
    var jsonData = wine.data;
    var dataBlob = {},
        sectionIDs = [],
        rowIDs = [],
        cars = [];

    for (var i = 0; i < jsonData.length; i++) {
      sectionIDs.push(i);
      dataBlob[i] = jsonData[i].title;
      cars = jsonData[i].cars;
      rowIDs[i] = [];

      for (var j = 0; j < cars.length; j ++) {
        rowIDs[i].push(j);
        // 这是最恶心的地方，我们需要按照固定格式去填充输数据见后文，
        dataBlob[i + ":" + j] = cars[j];
      }
    }

    // 随后数据更新即可异步刷新UI
    this.setState({
      dataSource: this.state.dataSource.cloneWithRowsAndSections(dataBlob, sectionIDs, rowIDs)
    });
  }
```

之所以说恶心，主要是因为我们要实现ListView的数据刷新，需要格式化数据格式按照`sectionID:rowID`的方式才能被ListView的黏性属性特性所识别，数据是下属一个sectionID才会被归纳为一个组中，具体数据样式如下所示，
```json
{
  "sectionID1" : "2333",
  "sectionID1:rowID1" : "2333",
  "sectionID1:rowID2" : "2333",
  "sectionID2" : "2333",
  "sectionID2:rowID1" : "233"
}
```

一些其它的小细节和CSS就不放出来了，项目细节见文末，po一张吸顶完成图

<img src="https://i.loli.net/2018/02/14/5a843cb9db199.gif" width = "40%" height = "40%" align=center />


在RN中实现九宫格，你可以认为是iOS中collectionView，主要还是用到了ListView组件，只不过修改的内容在ListView的CSS中，核心ListView CSS如下所示，

```CSS
listViewStyle: {
    flexDirection: 'row',
    flexWrap: 'wrap'
  },
```

剩下的实现方法与在iOS中手撸九宫格效果是一样的，比如根据当前屏幕宽度算每个Cell的左右边距，设置个数等等。

tabBar是我最喜欢RN的地方（实现是太方便了🙂），因为tabBar在RN中有平台差异性，在此我们先用RN本身提供的tabBarIOS进行实现，先来看看到底是怎么写的，
```js
render() {
    return (
      <View style={styles.container}>
        <View style={styles.headViewStyle}>
          <Text style={{fontSize: 25, marginTop: 10}}>PJHubs</Text>
        </View>
        <TabBarIOS
          barTintColor= "black"
          style={{flex: 1}}
        >
          <TabBarIOS.Item
            systemIcon= "contacts"
            badge= "3"
            selected={this.state.selectedTabBarItem == "home"}
            onPress={() => {this.setState({selectedTabBarItem: "home"})}}
          >
            <View style={[styles.commonViewStyle, {backgroundColor: "red"}]}>
              <Text style={styles.commonTextStyle}>首页</Text>
            </View>
          </TabBarIOS.Item>

          <TabBarIOS.Item
            systemIcon= "bookmarks"
            selected={this.state.selectedTabBarItem == "second"}
            onPress={() => {this.setState({selectedTabBarItem: "second"})}}
          >
            <View style={[styles.commonViewStyle, {backgroundColor: "blue"}]}>
              <Text style={styles.commonTextStyle}>首页</Text>
            </View>
          </TabBarIOS.Item>

          <TabBarIOS.Item
            systemIcon= "downloads"
              selected={this.state.selectedTabBarItem == "third"}
              onPress={() => {this.setState({selectedTabBarItem: "third"})}}
          >
            <View style={[styles.commonViewStyle, {backgroundColor: "green"}]}>
              <Text style={styles.commonTextStyle}>首页</Text>
            </View>
          </TabBarIOS.Item>

          <TabBarIOS.Item
            systemIcon= "search"
                selected={this.state.selectedTabBarItem == "four"}
                onPress={() => {this.setState({selectedTabBarItem: "four"})}}
          >
            <View style={[styles.commonViewStyle, {backgroundColor: "purple"}]}>
              <Text style={styles.commonTextStyle}>首页</Text>
            </View>
          </TabBarIOS.Item>
        </TabBarIOS>
      </View>
    );
  }
```

从以上代码中可以看到，主要是就是`tabBarIOS`和`tabBarIOS.Item`这两个组件，关于每个横向的tabBar中要显示什么，直接在tabBarItem闭合标签中写明即可。需要注意的地方是，RN中tabBar比iOS原生TabBar多了一步手动选中的操作（也可以认为是更加简明🙂），我们需要先指明一个第一次进来选中的tab，
```js
constructor(props) {
    super(props);
    this.state = {
      selectedTabBarItem: "home"
    };
  }
```

随后，如上图tabBar创建代码所示，根据`selected`属性和`onPress`回调函数进行tab的选中判断和变量赋值。实际上Native的tabBar选中判断方法也如此这般的操作，只不过我还是喜欢Native的做法🙂。

<img src="http://7xszq8.com1.z0.glb.clouddn.com/tarbar.gif" width = "40%" height = "40%" align=center />

[详细代码见工程，learnRN3、learnRN_ListView](https://github.com/windstormeye/ReactNativePractices)
