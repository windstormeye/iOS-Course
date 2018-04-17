本系列文章为记录iBistu 4.0各个模块开发中进行的思考、设计和编码总结，供同学们参考。

---

# 数据源

## 获取黄页部门列表数据
**接口：**
ibistu/_table/module_yellowpage?filter=isDisplay%3D1&offset=1&group=department
**请求方法：**get
**参数：**无
**示例请求成功返回值：**
```json
{
    "resource": [
        {
            "id": 94,
            "name": "研究生部（党委研究生工作部）",
            "telephone": "1",
            "department": 10,
            "isDisplay": true
        },
        {
            "id": 102,
            "name": "人事处",
            "telephone": "1",
            "department": 11,
            "isDisplay": true
        }
    ]
}
```

通过以上数据源实例，我们能够发现返回的“resource”是个数组，我们可以把其转化为一个数组对象进行使用，并且我们还能够从数据源中发现，每个数组中的单个“对象”提供的信息非常有限，能够提取的出来的信息只有“name”能够用上。

这也就代表了，我们如果想要在“黄页”上玩出花来是一件非常困难的事情。

## 获取黄页某一部门下的电话号码数据
**接口：**http://api.iflab.org/api/v2/ibistu/_table/module_yellowpage
**请求方法：**get
**参数：**
offset：固定参数，值为1
filter：固定前缀department=，值为黄页接口1返回的数据中的department字段值
**示例：**获取研究生工作办公室的电话号码：（此处参数值为：department=10）：http://api.iflab.org/api/v2/ibistu/_table/module_yellowpage?offset=1&filter=department=10

**示例请求成功返回值：**
```json
{
  "resource": [
    {
      "id": 95,
      "name": "主任室",
      "telephone": "82426837",
      "department": 10,
      "isDisplay": true
    },
    {
      "id": 97,
      "name": "副主任室",
      "telephone": "82426097",
      "department": 10,
      "isDisplay": true
    },
    {
      "id": 101,
      "name": "学位与学科建设/行政办公室",
      "telephone": "82426838",
      "department": 10,
      "isDisplay": true
    }
  ]
}
```

从部门详情中可以看到，跟之前的数据源格式一模一样，不过我们可以利用的信息多了“telephone”字段。

因此，综上所诉，为了顾及简单明了的显示规则，对此不作过多的修改，直接使用tableView作为所有部门和各个部门详情的数据源载体。

# 设计

<div align="center">    
<img src="http://7xszq8.com1.z0.glb.clouddn.com/ww.png" width = "100%" height = "100%" align=center />
</div>

部门列表我们采用tableView去展示数据源，点击部门列表cell将会push进对应部门详情，同样在部门详情也使用tableView去展示数据源，但点击部门详情cell时我们将会调起原生电话API，拨打展示出的对应部门电话号码。

# 编码

1. [详细编码见工程](https://github.com/ifLab/iCampus-iOS)

2. 在部门列表顶部有一个搜索框，要实现模糊搜索，因为受到API的限制并没有单独开一个接口去接收textField文本内容变化后而返回新的数据。

    因此，我们的做法只能查找静态数据源，相同的点还是在搜索框的文本内容改变代理方法中进行处理。在处理体中，因为数据源都已经异步拿到了，虽然有异步的时间差，但一般来说没有用户会在列表数据源没出来直接进行搜索的。

    在端上直接进行模糊搜索有两种做法，一是通过NSRegularExpression类创建正则表达式后再进行匹配，二是使用NSPredicate类创建谓词匹配实例，关于这两部分内容大家可以参考[这篇文章](https://www.cnblogs.com/pruple/p/5865208.html)

    ```Objc
    - (void)searchBar:(UISearchBar *)searchBar textDidChange:(NSString *)searchText; {
        // 使用谓词匹配
        NSPredicate *preicate = [NSPredicate predicateWithFormat:@"SELF CONTAINS[c] %@", searchText];
        if (_kSearchArr != nil) {
            [_kSearchArr removeAllObjects];
        }
        for (NSDictionary *dict in _dataArr) {
            NSString *str = dict[@"name"];
            if ([preicate evaluateWithObject:str]) {
                [_kSearchArr addObject:dict];
            }
        }
        [self reloadData];
    }
    ```
    在黄页搜索框中，我们采取了谓词的CONTAINS做了模糊搜索，谓词匹配的使用比较简单直接调用evaluateWithObject方法，传入str，只要str中有一个字符与searchText中的任何字符匹配即可。注意，此处并不需要对中文字符做编码集的替换，因为两者对比的都是NSString类型，NSString默认是Unicode编码（如果没记错的话）所以没必要可以直接对中文字符进行比对。

<img src="http://7xszq8.com1.z0.glb.clouddn.com/%E9%BB%84%E9%A1%B51.gif" width = "40%" height = "40%" align=center />

# 总结

1. 黄页模块是iBistu for iOS整体最经典的部分，运用了tableView的基本思想和操作，可以快速帮助新手快速切入iBistu开发。
2. 此处有一个待优化的地方。每次点击一个部门详情后，都会拉一次数据，这种做法针对咨询类产品是正确的，但是针对这种黄页类产品，数据更新允许滞后，这部分应该做缓存，只有每次在所有部门列表下拉时才进行数据源的全部更新。


[下篇：iBistu4-0（地图）](./iBistu4-0（地图）.md)
