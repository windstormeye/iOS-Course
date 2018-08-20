## UICollectionView 的使用、注意点和优化

### 让 UICollectionView 进行滚动
这是我今天刚遇到的问题，虽然 `UICollectionView` 是继承 `UIScrollView` 的，而且也有 `contentSize` 属性，但是对其设置 `CGSize` 并无效果，因为我已经使用了 `UICollectionViewFlowLayout` 布局风格，当 `cell` 数量超出一屏时，将会自动拓展 `contentSize` ，我们需要做的就是 `collectionView?.alwaysBounceVertical = true` 