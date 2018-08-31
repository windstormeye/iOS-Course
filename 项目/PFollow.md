## PFollow
在这一系列文章中主要记录了我在开发 PFollow 过程中遇到的问题和总结。

### 解决点击用户定位蓝点出现 callout 的问题：
func mapView(_ mapView: MAMapView!, didAddAnnotationViews views: [Any]!) {
    for view in views {
        let v = view as! MAAnnotationView
        if v.tag == 0 {
            v.isUserInteractionEnabled = false
        }
    }
}

### 获取当前地图可视区域中的所有大头针
哇，高德地图的文档真的太垃圾了。虽然功能确实挺全的，但是很多东西都没有指导性，获取当前地图可视区域的所有大头针，搞了好久都没有弄明白怎么获取这个可视区域到底怎么获取，幸亏搜到了 `MKMapView` 的相同方法，直接丢个 `mapView.visibleMapRect` 就完事了。😅

### 分块加载
之前对于根据用户缩放地图等级进行大头针图片的切换用了一个非常不好的办法，今天狠下心来逼自己必须改进哈哈哈。

具体的思考流程是这样的，从以往使用地图经验来看，不管是哪一家地图产品都是分块加载，只有当用户滑动到了地图上的某块为加载过的区域才进行地图的加载，如果滑到了已经加载过的区域则不会再进行地图渲染，因为之前的数据已经 `load` 到了内存中。

所以正是因为这一点，我坚决的认为高德是一定能够完成根据用户缩放地图的等级来进行切换大头针的，经过一番纠结后发现能够拿到目前屏幕可视区域中的所有标注点集合 `Set<AnyHashable>` 而且还是包括了用户定位的小蓝点在内。

因此做法是，首先在 `mapDidZoomByUser` 用户缩放地图完毕后的回调中判断 `mapView.zommLevel` ，在我们需要的数值内通过 `mapView.annotations(in: <#T##MAMapRect#>)` 方法获取到上文所说的 `Set<AnyHashable>` ，当然此时的 `Set` 是无法使用的，因为是 `Any` 类型，而且里边还有 `MAUserLocation` ，此时可以使用 `Swift` 的精华函数式编程的一个方法 `filter`，如下所示：

```Swift
let annotationSet = mapView.annotations(in: mapView.visibleMapRect).filter { (item) in
                !(item is MAUserLocation) } as! Set<MAPointAnnotation>
```

当然，这种写法还不是最优的，但这是我第一次使用 `FP` 的一些思想，还没上道，但是就上边这两行代码真的比之前那种特别傻的 `for-in` 循环好太多了。😄

接着拿到筛出来的 `MAAnnotation` 集合转为 `Array` 遍历（没找到怎么给 `Set` 遍历），然后就根据 `longitude` 和 `latitude` 在之前创建的自定义大头针数组中 找到分别判断该大头针的 `.image` 是否为想要改变的大头针，不是再修改。

因为是分块加载计算，所以并不会像之前写的那种特别傻的方法，直接全部移除再添加，改进后的这个只是在当前用户屏幕可视区域内进行移除和添加，再加上这个缩放初始化为 15 ，缩放要小于 12.8 才改小的大头针图片，所以实际上并不需要占用多大的计算资源，但是算法还是需要改进，现在是两个 `for` 循环，O(n^2) ，有点伤。