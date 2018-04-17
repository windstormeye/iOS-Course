---
title: iBistu4.0（失物）
date: 2018-01-25 10:33:21
tags:
- iOS
- iBistu
---

本系列文章为记录iBistu 4.0各个模块开发中进行的思考、设计和编码总结，供同学们参考。

---

# 数据源
## 获取失物招领列表数据
**接口：**http://api.iflab.org/api/v2/ibistu/_table/module_lost_found?limit=10&order=createTime%20desc
**请求方法：**get
**参数：**

`offset：`必需参数，招领信息（页数-1）的10倍，当offset为0时，返回的是第一页的数据，offset为10时，返回的是第二页，20时返回的是第三页，依次类推。
`filter：`必需参数，只有两个值，值为“isFound=false”时，返回普通列表数据；当值为“(isFound=false)And(author=用户名)”时，返回的是该用户发布的所有招领信息

**示例：**获取第2页的用户名为mphone的已发布信息列表，：（此处参数值为：offset=10，filter=(isFound=false)And(author=mphone)，如果想获取普通列表，则filter=isFound=false）：http://api.iflab.org/api/v2/ibistu/_table/module_lost_found?limit=10&order=createTime%20desc&offset=10&filter=isFound=false

示例请求成功返回值：
```json
{
    "resource": [
        {
            "id": 86,
            "details": "捡到一堆小玩意儿",
            "createTime": "2016-10-13 13:26:30",
            "author": "mphone",
            "phone": "13622251463",
            "isFound": false,
            "imgUrlList": "[{\"url\":\"files/ibistu/lost_found/image/14763363892752229.jpg\"},{\"url\":\"files/ibistu/lost_found/image/14763363331344112.jpg\"}]"
        },
        {
            "id": 85,
            "details": "捡到一盒餐具，不知道是谁的，如图所示",
            "createTime": "2016-10-13 13:25:33",
            "author": "mphone",
            "phone": "13688881425",
            "isFound": false,
            "imgUrlList": "[{\"url\":\"files/ibistu/lost_found/image/14763363331344112.jpg\"}]"
        }
    ]
}
```

从获取失物招领数据源中可以看到，我们可以拿到该条失物招领的“描述信息”、“发起者”、“创建时间”、“发起者联系方式”、“失物图集”。并且还是个数组源，给我们的暗示可以采用tableView作为内容载体，并且“失物图集”也是个数组源。

## 发布新招领信息
**接口：**http://api.iflab.org/api/v2/ibistu/_table/module_lost_found
**请求方法：**post
**参数：**
`details`:失物招领细节；
`author`：失物发起者；
`phone`：失物发起者联系方式；
`imgUrlList`：失物图集相对地址。（下文讲解）
**请求体：**(其中imgUrlList中的url为:“files/上传图片成功后返回的path”)
```json
[
   {
       "details": "捡到一盒餐具，不知道是谁的，如图所示",
       "author": "mphone",
       "phone": "13688881425",
       "imgUrlList": "[{\"url\":\"files/ibistu/lost_found/image/14763363892752229.jpg\"},{\"url\":\"files/ibistu/lost_found/image/14763363331344112.jpg\"}]"
   }
]
示例请求成功返回值：（可忽略）
{
  "resource": [
    {
      "id": 93
    }
  ]
}
```

该API需要我们对其进行POST请求，关于HTTP请求方式POST和GET的区别可以参考知乎上的[这个问题](https://www.zhihu.com/question/28586791)，我们需要构造入口先获取到失物和用户信息详情。

## 上传图片（多图上传）
**接口：**http://api.iflab.org/api/v2/files/ibistu/lost_found/image/
**请求方法：**post
**请求体：**（上传图片需要将图片转换为base64格式的编码字符串，然后作为content）
**请求参数：**
`name`：图片文件名（随意，目前我采用的是当前时间戳）
`type`：统一默认为file；
`is_base64`：统一默认为true；
`content`：base64对压缩完的图片进行编码。
```json
{
    "resource": [
        {
            "name": "filename1.jpg",
            "type": "file",
            "is_base64": true,
            "content": "base64字符串"
        },
        {
            "name": "filename2.jpg",
            "type": "file",
            "is_base64": true,
            "content": "base64字符串"
        },
        ...,
        {
            "name": "filename3.jpg",
            "type": "file",
            "is_base64": true,
            "content": "base64字符串"
        }
    ]
}
```
**示例请求成功返回值：**（上传成功后文件的真实存储路径为：“http://api.iflab.org/api/v2/files/上传成功后返回的path值”）
```json
{
 "resource": [{
   "name": "filename1.jpg",
   "type": "file",
   "path": "图片存放的相对路径"
 }
 ,{
   "name": "filename2.jpg",
   "type": "file",
   "path": "图片存放的相对路径"
 }
 ...
 ,
 {
   "name": "filename3.jpg",
   "type": "file",
   "path": "图片存放的相对路径"
 }]
}
```
要求我们要把需要上传的图片用base64进行编码，千万记得先进行压缩，要不然base64对图片进行编码完后的数据量会非常非常非常大😓，因为base64是对其进行像素转化编码，分辨率越高，编码后得到的加密字符串也就越长，耗费的网络资源也就越高。

关于iOS中对图片的压缩，我们可以使用`drawInRect`重新对该张图片进行规定大小的绘制拿到降低size的图片，或者使用一些第三方库即可（我没找到合适的）

```ObjC
-(UIImage *) imageCompress:(CGFloat)targetWidth {
    CGFloat width = self.size.width;
    CGFloat height = self.size.height;
    CGFloat targetHeight = (targetWidth / width) * height;
    UIGraphicsBeginImageContext(CGSizeMake(targetWidth, targetHeight));
    [self drawInRect:CGRectMake(0,0,targetWidth,  targetHeight)];
    UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newImage;
}
```

以上是UIImage的分类方法，跟失物详情获取分享图片的做法是一样的，需要开启当前图片上下文进行缩放的size绘制，绘制完成后拿到一个压缩后的图片。之所以能够这么做是因为，我们把原始图片按照一定缩小比例重绘拿到新的图片后，并未对UIImageView进行大小缩放，还是原先的大小，这就导致用大框（UIImageView）作为了小图（UIImage）的载体。肯定还有其它的压缩方式，而且肯定也还有很多无损压缩方式，这是一种投机的有损压缩（不知道这么说大家明白没）

# 设计

<img src="http://7xszq8.com1.z0.glb.clouddn.com/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png" width = "100%" height = "100%" align=center />

根据之前的说法，我们根据失物列表数据源的特性使用tableView作为载体，点击tableViewCell后跳转到对应的失物详情中。在失物列表，除了“失物图集”外其它信息都是简略的，进入到对应的失物招领详情后才能够获取到所有详细信息，并且可直接调起拨打电话和发送短信。

# 编码

1. [详细编码见工程](https://github.com/ifLab/iCampus-iOS)

2. 失物列表的“失物图集”部分3.3中的设想是scrollView直接把图集左右摆平，然后把imageView的用户点击事件打开，点击图集中的某一张图片后再放大即可。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4895.PNG" width = "50%" height = "50%" align=center />

3. 失物详情是4.0才加入的新模块，原本只能在失物列表中查看和拨打电话，这就导致了用户在失物列表中查看某一条失物招领信息时并不能快速的获取有效信息，为了达到“一站式”的浏览失物，我们不但在3.3能够直接在APP中拨打电话的情况下还继续添加了直接发送短信、分享失物到主流社交平台上。（虽然一点难度没有emmm）
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4896.PNG" width = "50%" height = "50%" align=center />

4. 失物详情的分享部分采取的是使用Core Graphics框架相关API。失物详情整体用scrollView作为载体，此时我们把scrollView作为绘制的母体，获取其size作为绘制画布的大小和内容，最后拿到一张供分享的图片。（不知道为啥配色又没了。）核心实现如下，首先要先设置要开启绘图上下文的一些选项，比如绘图区域、是否透明、缩放等等。其次我们还要设置开启绘图的内容，把失物详情scrollView作为绘图内容，最后关闭绘图上下文。

    ```ObjC
        UIGraphicsBeginImageContextWithOptions(scrollView.contentSize, YES, 0.0);
        [scrollView.layer renderInContext:UIGraphicsGetCurrentContext()];//renderInContext呈现接受者及其子范围到指定的上下文
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();//返回一个基于当前图形上下文的图片
        UIGraphicsEndImageContext();
    ```
    <img src="http://7xszq8.com1.z0.glb.clouddn.com/1233.jpg" width = "50%" height = "50%" align=center />

5. 在“发布新失物”部分，需要直接提取当前登录用户名，保证失物发布的真实性，用户手动补充联系电话、描述信息和失物图集，且默认至少发布一张失物照片。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4894.PNG" width = "50%" height = "50%" align=center />

# 总结

1. 失物模块整体上可以说是黄页模块的进阶，黄页模块整体都是tableView一路走到底，但是失物杂糅了tableView、scrollView和一些自定义部分页面，适合新手切入学习。

2. 失物模块在4.0原本是想换成瀑布流样式的，但由于时间因素并未能持续推进。


