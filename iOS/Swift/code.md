# 有趣的代码段

### `Bool` 怎么转 `Int`
```swift
return boolValue ? 1 : 0
```

### `Int` 怎么转 `Bool`
```swift
return intValue != 0
// or
(intValue != 0)
```

### Swift 做域
```swift
extension Tracker {
    struct SVE {}
}

extension Tracker.SVE {
    func xxx() {

    }
}

Tracker.SVE.xxx()
```

### `@PropertyWrapper` 如何使用

详见：[https://medium.com/@EvangelistApps/property-wrappers-in-swift-51cee87e2c32](https://medium.com/@EvangelistApps/property-wrappers-in-swift-51cee87e2c32)

简单来说其本质也是方法，只不过这个方法是编译器确认，能够在写代码的时候就调用，加入了检查环节，保证未开始构建时就能够发现问题。