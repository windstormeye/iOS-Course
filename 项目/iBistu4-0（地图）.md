---
title: iBistu4.0（地图）
date: 2018-01-26 13:31:16
tags:
- iBistu
- iOS
---

# 数据源

## 获取校区位置数据
**接口: **http://api.iflab.org/api/v2/ibistu/_table/module_map
**请求方法：**get
**参数：**无
**示例请求成功返回值：**
```json
 {
 "resource": [
   {
     "id": 5,
     "areaName": "小营校区",
     "areaAddress": "北京市海淀区清河小营东路12号",
     "zipCode": "100192",
     "longitude": 40.036,
     "latitude": 116.349,
     "zoom": 10
   },
   {
     "id": 6,
     "areaName": "健翔桥校区",
     "areaAddress": "北京市朝阳区北四环中路35号",
     "zipCode": "100101",
     "longitude": 39.988,
     "latitude": 116.392,
     "zoom": 10
   },
   {
     "id": 9,
     "areaName": "酒仙桥校区",
     "areaAddress": "北京市朝阳区酒仙桥六街坊1号院",
     "zipCode": "100016  　　",
     "longitude": 39.963,
     "latitude": 116.49,
     "zoom": 10
   }
 ]
}
```

毕竟是地图模块，因此是一定要使用Map，为了降低打包后的额外增长体积，考虑使用原生MapKit进行开发。使用MapKit呈现具体的地理位置最重要的就是经纬度，从数据源中的`longitude`（经度）和`latitude`（纬度）中可以拿到。

在iOS中的Map中有“大头针”的概念，而我们给大头针经纬度也只是把“大头针”的位置在Map上给标记出来，实际上一个“大头针”代表着什么信息我们并不知道。此时，数据源中的`areaName`和`areaAddress`字段数据就可以较好的给用户一个“大头针”专属提示。

## 获取班车数据
**接口：**http://api.iflab.org/api/v2/ibistu/_table/module_bus
**请求方法：**get
**参数：**无
**示例请求成功返回值：**
```json
{
    "resource": [
        {
            "id": 5,
            "busName": "西线班车",
            "busType": "通勤班车",
            "departureTime": "06:55:00",
            "returnTime": "17:10:00",
            "busLine": "[{"station":"公主坟","arrivalTime":"06: 55"},{"station":"当代商城","arrivalTime":"07: 05"},{"station":"蓝旗营清华西南门","arrivalTime":"07: 15"},{"station":"小营校区","arrivalTime":"07: 45"},{"station":"小营校区","arrivalTime":"17: 10"},{"station":"蓝旗营清华西南门","arrivalTime":"17: 35"},{"station":"当代商城","arrivalTime":"17: 50"},{"station":"公主坟","arrivalTime":"18: 20"}]",
            "busIntro": "西线班车"
        },
        {
            "id": 11,
            "busName": "望京-健翔桥",
            "busType": "通勤班车",
            "departureTime": "07:15:00",
            "returnTime": "17:10:00",
            "busLine": "[{"station":"望兴园","arrivalTime":"07: 15"},{"station":"望京花园","arrivalTime":"07: 17"},{"station":"健翔桥校区","arrivalTime":"07: 45"},{"station":"健翔桥校区","arrivalTime":"17: 10"},{"station":"望京花园","arrivalTime":"17: 45"},{"station":"望兴园","arrivalTime":"17: 47"}]",
            "busIntro": "望京-健翔桥"
        },
        {
            "id": 12,
            "busName": "小营-清河校区南门-健翔桥",
            "busType": "教学班车",
            "departureTime": "09:50:00",
            "returnTime": "",
            "busLine": "[{"station":"小营校区","arrivalTime":"09: 50"},{"station":"清河小区南门","arrivalTime":"09: 51"},{"station":"健翔桥校区","arrivalTime":"10: 35"}]",
            "busIntro": "小营-清河小区南门-健翔桥"
        },
        {
            "id": 17,
            "busName": "健翔桥-小营",
            "busType": "教学班车",
            "departureTime": "09:50:00",
            "returnTime": "",
            "busLine": "[{"station":"健翔桥校区","arrivalTime":"09: 50"},{"station":"小营校区","arrivalTime":"10: 10"}]",
            "busIntro": "健翔桥-小营"
        }
    ]
}
```

为什么在地图模块中也要引入班车呢？在3.3中班车和地图独占一方，但在4.0中我们把班车和地图杂糅在了一块，大家从数据源中可以看到每个班车列表排班中都有“地点”属性，而地图模块就是“地点”属性集合的载体，因此，我们做出了这个决定。

班车模块的数据源中还有“通勤班车”和“教学班车”两种类型，需要我们对班车类型从用户体验角度出发做区别。再加上各个车次有非常明显的排班时间和到达地点，这也给了我们一个暗示，可以对“busLine”字段做一个类似于“时间轴”的设计。

## 导航

这个功能在后端文档上没有体现出来，其实也不需要写啥问题。我们想到达到的效果在Map上选择一个“大头针”，然后再点击“导航”按钮，即可开启导航功能。


# 设计

<img src="http://7xszq8.com1.z0.glb.clouddn.com/ww%20%281%29.png" width = "100%" height = "100%" align=center />

我们设想在把leftBarButton作为班车入口，rightBarButton作为导航入口，分别使用push的方式切入。而班车模块，还是使用了tableView作为数据源载体，并且把每俩班车的来回时间段都做二次push，重新丢入一个VC中做呈现。

导航功能我们天真的认为用户应该不会把原生地图App删除，默认直接跳进原生MapApp🙂。


# 编码

1. [详细编码见工程](https://github.com/ifLab/iCampus-iOS)

2. 在Map的改动部分，3.3是原生的大头针，到了4.0我们认为大头针太丑，遂想改动为自定义的大头针，用校标作为大头针logo。但问题又来了，因为在素材上没法保证高清和风格统一，所以最后的解决办法只能又撤回使用大头针，令人非常意外的是在iOS 11上的大头针出人意料的好看。远没有iOS 10中的那般突兀。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4897.PNG" width = "50%" height = "50%" align=center />

    还有一点非常出乎我的意料，当我们给大头针设置好相关的提示信息后放大地图点击大头针后可以发现，效果真的是超赞👍。虽然没有体现出“差异性”，但比起iOS 10那令人尴尬的大头针效果真的好很多。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_347EC07E17F3-1.jpeg" width = "50%" height = "50%" align=center />
    iOS 10地图大头针样式
<img src="http://7xszq8.com1.z0.glb.clouddn.com/%E5%9C%B0%E5%9B%BE1.png" width = "40%" height = "40%" align=center />

3. 校车模块的入口在map VC的leftBarButton。在3.3中因为不但要熟悉流程而且还要出活，因此没有在UI方面下多大功夫，用的基本上之前的素材，因此整体呈现出来的效果也不够理想，如下所示：
<img src="http://7xszq8.com1.z0.glb.clouddn.com/%E6%A0%A1%E8%BD%A61.png" width = "40%" height = "40%" align=center />

    大家可以看到效果实在是不佳，而且还有莫名的素材错位。😓。从用户整体的角度来看比较尴尬，再加上3.3本身也是一个产品过渡期，部分模块产生了一种不上不下的中间位置，没有一种“大刀挥舞”的做事快感，反而多了“婆婆妈妈”的细节。

    以下是4.0的校车模块，我们弱化掉了3.3中校车模块中其它元素的影响，比如“往、返”icon等，对“班车类型”做了重点区分，校车模块也算是一个咨询类模块吧，最担心的问题应该就是怕“找错信息”，因此我们想让用户目标聚集在先确定“班车类型”，随后才接着展开下一步操作。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4898.PNG" width = "40%" height = "40%" align=center />

    而点击进入某一条班次后的实际编码过程也遇到了很多问题，之前的做法是通过使用“色块”的方式来区分往返车次，但问题就出在这大量使用的“色块”上，原意是用于区分往返车次，但实际上过于抢眼，看班车详情不但是想要去看到什么时间出发、什么时间返回，其实更重要是还要知道中途都都在哪些时间点经过哪些地方，因此，我们在实际编码中前前后后做了以下参考。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/9E5DAEF9-BBB7-4CB1-8F76-EA21750F3B33-3465-000002750393AA0B_tmp.JPG" width = "40%" height = "40%" align=center />
<img src="http://7xszq8.com1.z0.glb.clouddn.com/3501FE1E-CC8D-453D-B656-39E6116A69B3-3465-0000027519F5A751_tmp.JPG" width = "40%" height = "40%" align=center />
<img src="http://7xszq8.com1.z0.glb.clouddn.com/BB2BE9FA-9AFA-421F-900F-49F090325509-3465-000002748C0B524A_tmp.JPG" width = "40%" height = "40%" align=center />

    其实最后的设计我还挺中意，但开发团队部分同学还是觉得不太合适，最终我们采取的做法如下，emmm这就是之前在先导篇中所说“前后闭包”设计，虽然我觉得还是哪有点怪怪的，但不管怎么说都比3.3中的清晰明了。
<img src="http://7xszq8.com1.z0.glb.clouddn.com/IMG_4899.PNG" width = "40%" height = "40%" align=center />

    如果各位同学有兴趣的话，可以去翻源码你会发现做“时间轴”的效果非常的有趣，嘿嘿嘿。

4. 在导航部分，之所以要限制用户必须要选择“大头针”后再进行导航操作就是因为跳转到自带地图应用需要传入地点、经纬度所以我们要先“告诉”代理方法数据源是什么才行。
```Objc
- (void)gothereWithAddress:(NSString *)address andLat:(NSString *)lat andLon:(NSString *)lon {
    //跳转系统地图
    CLLocationCoordinate2D loc = CLLocationCoordinate2DMake([lat doubleValue], [lon doubleValue]);
    MKMapItem *currentLocation = [MKMapItem mapItemForCurrentLocation];
    MKMapItem *toLocation = [[MKMapItem alloc] initWithPlacemark:[[MKPlacemark alloc] initWithCoordinate:loc addressDictionary:nil]];
    toLocation.name = address;
    [MKMapItem openMapsWithItems:@[currentLocation, toLocation]
                   launchOptions:@{
                                   MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving,
                                   MKLaunchOptionsShowsTrafficKey: [NSNumber numberWithBool:YES]
                                   }];
    return;
}
```
    而且在rightBarButton的点击事件中，我们要保证传入的调起系统地图代理方法地点的正确性，所以，我们要多加一个“北京信息科技大学”前缀，
```ObjC
- (void)rightItemClick {
    if (self.isSelectAnnotation) {

        NSDate *date = [NSDate date];
        NSDateFormatter *formatter = [[NSDateFormatter alloc]init];
        [formatter setDateFormat:@"MM-dd HH:mm:ss"];
        NSString *dateString = [formatter stringFromDate:date];
        NSDictionary *dic = @{
                              @"username" : [PJUser currentUser].first_name,
                              @"uploadtime" : dateString,
                              @"goto" : self.kAnnotationView.annotation.title
                              };
        [MobClick event:@"ibistu_map_nav" attributes:dic];

        [NSString stringWithFormat:@"%@", self.kAnnotationView];
        // 加上学校名称前缀
        [self gothereWithAddress:[NSString stringWithFormat:@"北京信息科技大学%@", self.kAnnotationView.annotation.title]
                          andLat:[NSString stringWithFormat:@"%f", self.kAnnotationView.annotation.coordinate.latitude]
                          andLon:[NSString stringWithFormat:@"%f", self.kAnnotationView.annotation.coordinate.longitude]];
    }else {
        [PJHUD showErrorWithStatus:@"请先选择地点"];
    }
}
```


# 总结

1. 地图模块是iOS开发初级进阶内容，适合熟悉了iOS开发基本内容后的同学进一步锻炼，且涉及到了数据源的二次定义和更多的自定义视图，对开发同学的产品设计和把控提出进一步要求。

2. 地图模块是3.3中我参与开发的个人因素带入最多的地方，融合了很多个人思想，在某些细节上考虑得必定不够周到和完善，这部分还是有一些细节可以持续提升，比如“时间轴”的改进。

3. 对于校车详情部分在4.0中主要有两个方向的设想，一是在校车列表中直接展开获取路线，二是同样采用push进行新的VC方式，但是路线都在地图上标记出来。我更倾向于第二种做法，但是由于时间关系（当时快期末了）未能持续推进。


[下篇：iBistu4-0（失物）](./iBistu4-0（失物）.md)
