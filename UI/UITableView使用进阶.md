1. `UITableView`可进行性能优化相关的点
  * Cell重用：
    - 数据源方法优化：创建一个静态变量重用ID，例如：`static NSString *cellID = @"cellID";`防止因为调用次数过多，static保证只创建一次，提高性能（感觉性能的提升可以忽略不记emmm）
    - 缓存池获取可重用Cell的两个方法：`dequeueReusableCellWithIdentifier:(NSString *)ID`会查询可重用Cell，若注册了原型Cell则能够查询到，否则为nil，故需要先判断`if(cell == nil)`
    - `dequeueReusableCellWithIdentifier:(NSString *)ID indexPath:(NSIndexPath *)indexPath`，使用之前必须通过SB/class进行可重用Cell的注册（registerNib/registerClass），不需要判断nil，一定会返回cell，若缓冲区Cell不存在，会使用原型Cell重新实例化一个新Cell。
  * 尽量使用一种类型的Cell：能够减少代码量，减小Nib文件的数量；保证只有一种类型的Cell，实际上App运后只有N个Cell，但若有M种Cell，则实际上运行最多却可能会是MxN 个。
  * 善用hidden隐藏subview：把所有不同类型的view都定义好，通过cell的枚举类型变量及hidden显示/隐藏不同类型的内容，因为在实际快速滑动中，单纯的显示/隐藏subview比实时创建快得多。
  * 提前计算并缓存Cell的高度。如果我们不预估行高，则优先调用`heightForRowAtIndexPath`获取每个Cell即将显示的高度，实际上就是要确定总的tableView.contenSize，最后才又接着调用`cellForRowAtIndexPath`，可以建一个frame模型，保存下提前计算好的cell高度。
  * 异步绘制：这是目前最火的tableView性能调优方法，新浪微博是这么做的，可以使用`ASDK`这个库进行。
  * tableView滑动时，按需加载：识别tableView静止或减速滑动结束后，异步加载，在快速滑动过程中，只按需加载目标方位内的Cell。
  * 避免大量使用图片缩放、颜色渐变、透明图层、CALayer特效（阴影）等操作，尽量显示大小刚好合适的图片资源。
