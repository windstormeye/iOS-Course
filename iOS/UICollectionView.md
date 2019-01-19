## UICollectionView 的使用、注意点和优化

### 让 UICollectionView 进行滚动
这是我今天刚遇到的问题，虽然 `UICollectionView` 是继承 `UIScrollView` 的，而且也有 `contentSize` 属性，但是对其设置 `CGSize` 并无效果，因为我已经使用了 `UICollectionViewFlowLayout` 布局风格，当 `cell` 数量超出一屏时，将会自动拓展 `contentSize` ，我们需要做的就是 `collectionView?.alwaysBounceVertical = true` 

### `UICollectionView` 执行 `reloadData` 方法时闪烁
在进行 Vary 2.0 开发时，发现了首页的 `UICollectionView` 在滑动时卡片的 `WKWebView` 在重用时会闪烁，一直以为是 `WKWebView` 的缓存问题，纠结了好几天，一直没搞懂 `WKWebView` 为啥会在重用时闪烁，也就是内容会跳动，而跳动的内容是基于之前载入过的内容。

来来回回好几次，尝试各种奇淫巧技都没能搞定 `WKWebView` 的闪烁问题，后来换个思路，开始查 `UITableView` 重用的时闪烁问题，边玩着 Vary 边纠结到底是哪出了问题呢？

突然发现在滑动卡片，并且正要切换时间时，卡片的内容居然都跳动并闪烁了！发现不对劲，开始跟 log，一条一条的 log 跟下来，发现执行了部分 `[collectionView reloadData]` 操作，开始隐约觉得这有问题！继续查阅资料，资料上写明，如果 `UICollectionView` 在执行 `reloadData` 操作时是会进行闪烁的！如果不需要这个闪烁的动画，可以关闭，代码如下所示：

```Objc
- (void)collectionViewReloadData {
    useWeakSelf
    [UIView setAnimationsEnabled:NO];
    
    [weakSelf.cardListView performBatchUpdates:^{
        [weakSelf.cardListView reloadData];
    } completion:^(BOOL finished) {
        [UIView setAnimationsEnabled:YES];
    }];
}
```

这么一来，滑动卡片时卡片再也不跳动闪烁了！哈哈～