## UI Dynamic
这篇文章中将记录我在使用 UI Dynamic 功能进行编码时遇到的问题。

### 简介
在使用 UIDynamic 时需要有一个全局的动画者，把我们需要发生的动态行为 `UIGravityBehavior` 进行统一管理，其实这跟做游戏也是一样的，首先需要创建出一个物理世界，然后在物理世界中加入我们要发生物理现象的节点，这些节点之间就会因为叠加上去的物理属性从而在相互影响时，因为叠加上去的物理属性，比如重力、弹力、摩擦力等等发生影响，从而模拟出真实物理世界中的效果。

### 创建一个动画者
因为这是管理全局动态的变量，所以它的生命周期要充满整个 vc ，相当于需要一个类内全局变量。动画者提供动力学相关的能力，同时还提供一个管理上下文。

`private var animator: UIDynamicAnimator?`

### 创建仿真行为
总共提供了 `UIGravityBehavior` 、`UICollisionBehavior` 、 `UIAttachmentBehavior` 、 `UISnapBehavior` 、`UIPushbehavior` 以及 `UIDynamicItemBehavior` 。

以下代码是场景一个重力仿真行为的缩略代码：

```Swift
let gravity = UIGravityBehavior()
let collisionBehavior = UICollisionBehavior()
collisionBehavior.collisionMode = .everything
collisionBehavior.translatesReferenceBoundsIntoBoundary = true
```

添加边界代码：

```Swift
collisionBehavior.addBoundary(withIdentifier: NSCopying, from: CGPoint() ,to:CGPoint())
```

仿真行为器可以创建多个，各个仿真行为器管控着不同的行为特性，比如重力仿真行为器 `UIGravityBehavior` 用于模拟重力相关的动力学效果， `UICollisionBehavior` 用于模拟物理世界中的边界相关动力学效果。

### 添加动力学元素
如果我们想要给一个仿真行为器添加动力学元素，则只需要

```Swift
behavior.addItem(entity)
```

### 发生效果
最后，如果我们想要让整套效果运行起来，需要把仿真行为添加到动画者中去，

```Swift
animator.addBehavior(behavior)
```

### 注意
我们需要自己维护谁和谁发生碰撞，也就是说，新增的动力学元素和哪个仿真行为下的动力学元素发生物理效果是需要我们自己维护的。



