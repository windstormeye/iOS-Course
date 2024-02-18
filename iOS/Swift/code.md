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