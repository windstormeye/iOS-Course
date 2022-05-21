# 数据源

## 获取新闻列表数据

**接口：** http://api.iflab.org/api/v2/newsapi/newslist

**请求方法：** get

**参数：**

`category`：必需参数，新闻分类，可选的值为"zhxw", "tpxw", "rcpy", "jxky", "whhd", "xyrw", "jlhz", "shfw", "mtgz"
`page`：必需参数，当前页数，从0开始，0表示第一页
**示例：**获取第5页的综合新闻：（此处参数值为：category=zhxw，page=4）：http://api.iflab.org/api/v2/newsapi/newslist?category=zhxw&page=5

**示例请求成功返回值：**
```json
[
    {
        "newsTitle": "我校召开2016年本科招生录取工作总结会",
        "newsTime": "2016-09-14",
        "newsLink": "http://news.bistu.edu.cn/zhxw/201609/t20160914_39069.html",
        "newsImage": "http://news.bistu.edu.cn/zhxw/201609/W020160914608238897988_135.jpg",
        "newsIntro": "9月14日上午，学校在小营校区召开2016年度本科生招生录取工作总结会。副校长许宝杰主持会议并做总结讲话，相关职能部门负责人及招生录取工作相关人员参加了会议。\n　　\n"
    },
    {
        "newsTitle": "学校领导走访校友企业“佰能蓝天”",
        "newsTime": "2016-09-13",
        "newsLink": "http://news.bistu.edu.cn/zhxw/201609/t20160914_39066.html",
        "newsImage": "",
        "newsIntro": "9月13日下午，校友会常务副会长、副校长韩秋实，校友会副会长、校长助理林国策，校友校史工作办公室、自动化学院相关领导及工作人员一行前往校友企业“北京佰能蓝天科技股份公司”调研，并祝贺“佰能蓝天”公司成...\n"
    },
    ...
    {
        "newsTitle": "【院长访谈】 李宁教授畅谈大类招生分流培养",
        "newsTime": "2016-09-12",
        "newsLink": "http://news.bistu.edu.cn/zhxw/201609/t20160912_39011.html",
        "newsImage": "http://news.bistu.edu.cn/zhxw/201609/W020160912316469392018_135.jpg",
        "newsIntro": "“大类招生、分流培养”作为一种新的人才培养模式正逐渐被国内外高校所采用。计算机学院是我校实行大类招生的试点学院。大类招生始于2014年，至今已有3年时间。\n"
    }
]
```

从数据源中我们可以看出，新闻模块数据源跟目前市面上的新闻资讯类App展示出的数据种类大致相同，有`newsTitle`、`newsTime`、`newsImage`、`newsIntro`等，所以此处给了我们一个暗示可以参考目前市面上已有的资讯类App的设计。

## 获取新闻详情
**接口：**http://api.iflab.org/api/v2/newsapi/newsdetail
**请求方法：**get
**参数：**
`link`：必需参数，新闻列表接口中返回的newsLink内容

**示例请求成功返回值：**
```json
{
    "title": "【院长访谈】 李宁教授畅谈大类招生分流培养",
    "time": "2016-09-12",
    "article": "　　“大类招生、分流培养”作为一种新的人才培养模式正逐渐被国内外高校所采用。计算机学院是我校实行大类招生的试点学院。大类招生始于2014年，...是需要改革的，而不是完全听命于受教育者和当前的就业市场，因而大类招生也同时考量着教育者和管理者的胆识和办学自信。（供稿：党委宣传部）",
    "imgList": [
        "http://news.bistu.edu.cn/zhxw/201609/W020160912316468133294.jpg",
        "http://news.bistu.edu.cn/zhxw/201609/W020160912316468133295.jpg",
        "http://news.bistu.edu.cn/zhxw/201609/W020160912316468133296.jpg"
    ]
}
```

新闻详情不是必须的，但是学校官网并没有做移动端适配，导致直接设置webView的requestURL出来的是的full size规格的页面。

新闻详情的数据源太长了，大家有兴趣的话可以参考iBistu的后端文档看看，其中有趣的是你会发现返回的新闻详情数据源实体并未是H5标签，而是字！符！串！，那么新闻中的图片去哪了？对！就是`imgList`字段，如果你真的去看了后端文档**新闻详情**部分的接口，你会发现那一大段的新闻详情实体介绍居然中间有回车换行符！！！这部分我是用的chrome的JSONVIEW格式化的JSON数据后发现的有几个可爱的“回车符”！

再仔细对比原新闻和3.3版本iBistu，emmm这几个换行符的位置就是要插入图片的位置，关于搞定iBistu新闻详情模块的细节可以参考之前为iBistu新闻做的微信小程序[开发总结](http://pjhubs.cn/2018/01/13/%E5%B0%8F%E7%A8%8B%E5%BA%8F%E5%88%9D%E6%8E%A2%EF%BC%88%E4%BA%8C%EF%BC%89/)，以下为节选，

>······我的乖乖，您看懂是什么意思了么？一堆回车符的地方就是要插入图片的地方。😱。所以我们要根据回车符出现的地方来判断是否应该插入图片，看了看iBistu的新闻详情部分的实现，确实是这么做的。


>但是更伤的问题来了，根据回车符我截断字符串在数组里，然后po出了内容，一看更呆了，图片在一个地方集中出现得越多，那么这块地方的换行符也就越多，换句话说，得根据回车符的多少来决定插入的图片数量。

>嗯，其实这还不是最伤的，按照这个思路弄完了以后，浏览了前面几条新闻，完美对上了，但是！！！到了后边的几条新闻就全乱了。该出现图片的地方没出现，不该出现的图片的地方空一大片

>原因是因为刚开始找到的规则是，三个换行放一张图片，如果当前区域放超过一张图片，比如说是两张图片，换行数则由三个变成了七个，也就是说，以三位基数，每多加两个换行多一张图片。

>但事实上不是这样啊！😭。后边几条新闻的换行数跟图片数对应关系完全不符合之前找到的，全乱了。这就非常的难受了。弄到最后，实在没办法，决定了如果是多张图片就只放一张😔。

>如果这部分你没能拿到真实数据好好研究一番的话，就算看了代码意义也不太大······

iBistu 4.0延用了之前3.3的实现，同样也是判断`\n`的出现，具体实现细节我们编码部分细说。

# 设计

<img src="http://7xszq8.com1.z0.glb.clouddn.com/ww%20%282%29.png" width = "100%" height = "100%" align=center />

仔细看喔，新闻详情居然是个tableView！！！我也不知道为啥之前设计的时候要用到tableView（这部分代码不是我写的，逃）

# 编码

1. [详细编码见工程](https://github.com/ifLab/iCampus-iOS)

2. 关于新闻分类的segementView，本着“多快好省”的理念🙂。直接用了[HMSegmentedControl](https://github.com/HeshamMegid/HMSegmentedControl)，使用起来非常简单，但是现在iBistu我要慢慢转变思维，以往都是直接看上哪个第三方库好用就二话不说直接拿来就开造，虽然能够让接手开发的同学快速出成果，但是iBistu毕竟是要体现出社团实际开发水平的一个东西，因此我有一个粗浅的想法，把iBistu用到的一些第三方库如果能够我们自己实现就自己实现。

    我们要不是说要取代这些用到的第三方库，而是要丢弃一些第三方库中所多余的东西，很多东西不是说它不好，而是说不合适，再加上现在社团的同学一届比一届强，我们有足够的实力造一些开源组件🙂。

3. 新闻列表，我们经历了几次变化，3.3中是沿袭了以往的设计，4.0刚开始进入开发阶段时我们采取的是简书的设计，做完了以后我发现图模糊而且设计上还是有些尴尬，比如字重的比例、文字的位置等

    在4.0快要结束的时候终于下定决心，抛弃copy for 简书的风格，改为copy for 知乎，是老版本的知乎（现在新版知乎也改了，改为了“字左图右”的样式，让用户更加专注于每个问题的文字内容而不是图片），从下图中可以看到，应该能够让大家更加喜欢喜欢看iBistu新闻吧！🙂
    <img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4888.PNG" width = "50%" height = "50%" align=center />

4. 新闻详情部分，使用到的是[DTCoreText](https://github.com/Cocoanetics/DTCoreText)来渲染HTML，因为这部分内容是3.3遗留下来的，然而事实上HTML能够直接用webView进行渲染的，并不知道这是为啥😓。po一段success块代码：
```ObjC
ICNewsDetail *news = [[ICNewsDetail alloc] init];
news.title = dic[@"title"];
news.creationTime = dic[@"time"];      news.pcURL = dic[@"imgList"];          news.body = [NSString stringWithFormat:@"<body><p>%@</p></body>", dic[@"article"]];
news.body = [news.body stringByReplacingOccurrencesOfString:@"\n" withString:@"</p><p>"];
news.body = [news.body stringByReplacingOccurrencesOfString:@"<p> </p>" withString:@"<p></p>"];
for (int i=0; i<news.pcURL.count; i++) {
    @try {
        news.body = [news.body stringByReplacingCharactersInRange:[news.body rangeOfString:@"<p></p><p></p>"] withString:[NSString stringWithFormat:@"<img src=\"%@\">", news.pcURL[i]]];
        } @catch (NSException *exception) {
            if ([exception.name isEqualToString:NSRangeException]) {
                break;
            }
        }
    }
    success(news);
```
可以看到是判断`\n`字符的出现来做`<p></p>`标签替换，然后再根据`<p></p>`的出现位置，插入`<img>`标签。
    <img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4889.PNG" width = "50%" height = "50%" align=center />
    新闻详情样式在4.1中会得到修改。

5. 新闻的分享功能采取的做法与失物模块的分享做法一直，新闻详情的数据源主体是tableView，tableView继承于scrollView，所以接下来的组佛就跟失物模块中的分享功能一样啦，有兴趣的同学可以自行转移阵地。

# 总结

1. 新闻模块为iBistu 4.0中技术难度相对较高的一部分，适合新手切入iBistu开发中的拔高模块。涉及到的难度主要是位于新闻详情（虽然这部分工作之前已经做好了。🙂）

2. 新闻模块的改动迭代次数较多而且每次迭代都是大改，虽然从整体上看难度不大，但很多地方都是团队开发同学一路来慢慢摸索，有时候费老大劲才做出来的效果，很多时候其它同学有了更好的思路就立马直接开始重新改了，现在总体看起来效果还算不错，但还算有改进的空间。


[下篇：iBistu4-0（黄页）](./iBistu4-0（黄页）.md)
