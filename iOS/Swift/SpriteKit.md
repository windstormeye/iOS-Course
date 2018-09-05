## Sprite Kit
这篇文章将记录我在使用 `Sprite Kit` 过程中遇到的问题和解决方法。

### 初始化一个场景

```Swift
private func initScene() {
        let scnView = SKView(frame: CGRect(x: 0, y: topView!.bottom,
                                           width: view.width, height: view.height - topView!.bottom))
        view.addSubview(scnView)
        
        let scene = PJPlacesAnnotationScene(size: scnView.frame.size)
        scene.scaleMode = .aspectFill
        scnView.presentScene(scene)
//        scnView.showsPhysics = true
        scnView.ignoresSiblingOrder = true
        
        scnView.showsFPS = true
        scnView.showsNodeCount = true
    }
```

### 初始化一个空心矩形
```Swift
bottleViwSprite = SKSpriteNode(imageNamed: "")
        bottleViwSprite?.size = CGSize()
        bottleViwSprite?.position = CGPoint()
        bottleViwSprite?.anchorPoint = CGPoint()
        bottleViwSprite?.physicsBody = SKPhysicsBody(edgeLoopFrom: CGRect())
        self.addChild(bottleViwSprite!)
```

### 初始化一个实心矩形
```Swift
let leftBox = SKSpriteNode(imageNamed: "")
        leftBox.size = CGSize()
        leftBox.position = CGPoint()
        leftBox.physicsBody = SKPhysicsBody(rectangleOf: CGSize())
        // 是否静止
        leftBox.physicsBody?.isDynamic = false
        bottleViwSprite?.addChild(leftBox)
```

### 初始化一个实心圆形
```Swift
let annotationSprite = SKSpriteNode(imageNamed: "")              
annotationSprite.position = CGPoint()
annotationSprite.size = CGSize()
annotationSprite.physicsBody = SKPhysicsBody(circleOfRadius: )
annotationSprite.physicsBody?.usesPreciseCollisionDetection = true
annotationSprite.scale(to: CGSize())
self.bottleViwSprite?.addChild(annotationSprite)
```

### 打开设备加速计并调整重力方向（倾斜）
```Swift
func startMonitoringAcceleration(){
    if motionManager.isAccelerometerAvailable {
        // 感应时间
        motionManager.accelerometerUpdateInterval = 0.2
        motionManager.startAccelerometerUpdates(to: OperationQueue.current!) { (data, error) in
            guard let accelerometerData = data else {
                return
            }
            let acceleration = accelerometerData.acceleration
            
            self.physicsWorld.gravity = CGVector(dx: acceleration.x * 9.8,
                                                    dy: acceleration.y * 9.8)
        }
    }
}
```