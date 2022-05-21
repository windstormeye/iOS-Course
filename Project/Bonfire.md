这篇文章主要记录我在开发Bonfire的过程中遇到的问题。

1. [CAShapeLayer的相关属性介绍](https://www.jianshu.com/p/98ff8012362a) 
2. 出现`dyld: Library not loaded: @rpath/libswiftAVFoundation.dylib`[解决方案](https://www.jianshu.com/p/46c3d65a996b)
3. `NSInvocation`使用。[参考文章](https://my.oschina.net/u/2340880/blog/398552)，当`SEL`需要传递多个参数时可以采用`NSInvocation`或者`block`

4. [基于responderChain的传值方式](https://casatwy.com/responder_chain_communication.html)

5. 屏蔽相机声音，最好是不要屏蔽，直接提示告诉用户开启静默模式之前应该打开静音，如果非要在关闭静音的模式下进行静默拍照，则：**使用和iOS系统拍照声音相反音波的音乐，在will方法中播放该音乐**。网上有例子。

6. `mapView.userTrackingMode`状态改变
```swift
func mapView(_ mapView: MKMapView, didChange mode: MKUserTrackingMode, animated: Bool) {
    // 当userTrackingMode发生改变时再把它改回去
}
```