# PJAlbumViewController
## 基于 `Photos` 框架，先来明确几个概念：

* `PHObject`：`Photos` 的资源集合和集合列表的抽象父类；
* `PHAssetCollection`：一个相册；
* `PHCollectionList`：一个包含多个相册的集合；
* `PHFetchResult`：
  * 作为 `PHAsset`（Live Photo）、`PHCollection`、`PHAssetCollection`、`PHCollectionList` 相关方法的返回结果对象；
  * 内容可动态加载，并不是直接把某个相册中的内容直接全部遍历出来，而是当需要一部分内容后才会去照片库中获取，这可以在处理大量结果时提供一个最佳性能；
 * 默认**线程安全**；
 * 缓存规则个人看法是利用了 `LRU`，但实际上是不是这么一回事有待考证。

以上内容均出自 `Developer Documentation`，并且内容不完整，因为官方文档中已经描述得十分清晰明朗，什么能做，什么不能做，做什么应该怎么做等情况都已列举，以上内容为摘要。

## 获取相册封面
### 相册种类
我们需要先列举出需要进行获取数据的相册类型，关于相册类型，网上已有很多资料，我的想法是：

```Swift
private func allAlbumCollection() -> [PHAssetCollection] {
    var collections = [PHAssetCollection]()
    let smartAlbums = PHAssetCollection.fetchAssetCollections(with: .smartAlbum,
                                                              subtype: .any,
                                                              options: nil) as! PHFetchResult<PHCollection>
    let userAlbums = PHCollectionList.fetchTopLevelUserCollections(with: nil)
    
    for album in [smartAlbums, userAlbums] {
        for s_i in 0..<album.count {
            let collection = album[s_i] as! PHAssetCollection
            let types: [PHAssetCollectionSubtype] = [.albumRegular, // 用户自己创建的相册
                                                      .smartAlbumPanoramas, // 全景
                                                      .smartAlbumScreenshots, // 屏幕截图
                                                      .smartAlbumUserLibrary, // 相机胶卷
                                                      .smartAlbumRecentlyAdded] // 最近添加
            if types.contains(collection.assetCollectionSubtype) {
                collections.append(collection)
            }
        }
    }
    return collections
}
```

### 获取相册中所有照片
`PHAssetCollection` 类型对象中涵盖了该相册中的所有属性，包括相册名、相片数等，我们需要对其进行遍历，拿到所有相片，我的做法是：
```Swift
/// 当前相册的所有 PJAsset
private func albumPHAssets(_ collection: PHAssetCollection) -> PHFetchResult<PHAsset> {
    let options = PHFetchOptions()
    // 只要照片，不要视频
    options.predicate = NSPredicate(format: "mediaType = %d", PHAssetMediaType.image.rawValue)
    // 按时间逆序
    options.sortDescriptors = [NSSortDescriptor.init(key: "creationDate", ascending: false)]
    let fetchResult = PHAsset.fetchAssets(in: collection, options: options)
    return fetchResult
}
```

### 照片资源的获取
拿到了存储照片的资源集合，下一步就是要对整个资源进行格式化统一为我们所需要的 `UIImage` 或 `Data`。需要注意的是，`requestImage` 这个方法是异步的！
```Swift
private func convertPHAssetToUIImage(asset: PHAsset, complateHandler: @escaping (_ photo: UIImage?) -> Void) {
    let coverSize = CGSize(width: 150, height: 150)
    let options = PHImageRequestOptions()
    options.isSynchronous = false
    options.deliveryMode = .fastFormat
    options.isNetworkAccessAllowed = true
    PHImageManager.default().requestImage(for: asset,
                                          targetSize: coverSize,
                                          contentMode: .default,
                                          options: options) { result, info in
                                            guard result != nil else { return complateHandler(nil) }
                                            complateHandler(result)
    }
}
```

### 统一获取
写好了以上两个方法，我们就可以来对其进行封装，我的写法是：
```Swift
/// 获取所有相册封面及照片数
func getAlbumCovers(complateHandler: @escaping (_ coverPhotos: [Photo], _ albumPhotosCounts: [Int]) -> Void) {
    let albumCollections = allAlbumCollection()
    var photos = [Photo]()
    var photosCount = [Int]()
    // 获取单张照片资源是异步过程，需要等待所有相册的封面图片一起 append 完后再统一通过逃逸闭包进行返回
    var c_index = 0
    for collection in albumCollections {
        let assets = albumPHAssets(collection)
        // 有些系统自带相册类型如果用户没有进行照片归类则会导致取到的相片数为0
        guard assets.count != 0 else {
            c_index += 1
            continue
        }
        
        photosCount.append(assets.count)
        
        var photo = Photo()
        photo.photoTitle = collection.localizedTitle
        convertPHAssetToUIImage(asset: assets[0]) { (photoImage) in
            photo.photoImage = photoImage
            photos.append(photo)
            c_index += 1
            
            if c_index == albumCollections.count - 1 {
                complateHandler(photos, photosCount)
            }
        }
    }
}
```