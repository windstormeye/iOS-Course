# UI

### UIKit 架构图
![UIKit 架构图](https://i.loli.net/2019/07/12/5d289ff297b8136683.png)

## 布局
### 界面布局（frame）
* `setNeedsLayout` 做标记，`runloop` 周期内重绘「下次 runloop」更新
* `layoutIfNeeded` 立即重绘做过标记的涂层「马上更新」
* `layoutSubviews` 不能直接调用，要通过 `setNeedsLayout` 间接调用

###  界面布局（auto layout）解释同上
* `setNeedsUpdateConstraints`
*`updateConstraintsIfNeeded`
* `updateConstraints`

### 布局约束计算
![布局约束计算](https://i.loli.net/2019/07/12/5d28a03db2f2442883.png)

![布局约束计算代码步骤](https://i.loli.net/2019/07/12/5d28a067b6d9b20237.png)

### Auto Layout 优化（避免规则扰动）
* 不要删除所有的约束（update 的时候）
* 若是一个静态约束，仅做一次添加操作即可
* 仅改变需要改变的约束
* 尽量不要做删除视图的操作，反之用 hide() 方法替代
* `intrinsicContentSize`
    - 如果能够确定 Size，直接返回
    - 如果不能，使用 `CGSize(width: UIView.noIntrinsicMetric, height: UIView.noIntrinsicMetric)`
* 不要过度使用 `systemLayoutSizeFitting()`

## 渲染
* CPU 渲染：`CoreGraphics`
* GPU 渲染：`OpenGL ES` 和 `Metal`
* CPU 很杂，可以什么都干
* GPU 只干渲染，还可以开启硬件加速

### CPU
* 对象创建、跳转和销毁
* 布局计算
* 文本计算
* 绘制（drawRect）。不能直接调用，不要滥用，会为视图分配一个寄宿图，寄宿图的像素尺寸等于视图大小 x contentScale 的值
* 图片解码
* 提交位图
### GPU
* 接收提交的纹理
* 纹理渲染
* 视图合成
* 光栅化

###  GPU 视图合成
多个子视图需要将视图纹理合成，将多个 View 的纹理拼接在一起，RGB 通道混合

### `drawRect:` 方法是 CPU 计算

### UIView 绘制
渲染是 CPU 和 GPU 共同协助完成的。在 `runloop` 中注册了一个 `Observer`，监听到 `BeforeWaiting` 和 `Exit` 事件进行渲染。

![UIView 绘制流程](https://i.loli.net/2019/07/12/5d28a101d105687163.png)


### UIView 渲染时机
Core Animation 在 `RubLoop` 中注册了一个 `Observer`，监听到 `BeforeWaiting` 和 `Exit` 事件进行渲染，`runloop` 的周期是 1/60s。

### 为什么会卡顿？
因为 `runloop` 1/60s 渲染一次，当 CPU 和 GPU 一次渲染耗时超过 16.7ms 那么就会来不及渲染，造成卡顿。

![为什么会卡顿？](https://i.loli.net/2019/07/12/5d28a1e709b8b22871.png)

### 卡顿监测
子线程在主线程处于 Before Waiting 时，ping 主线程，如果超时即发生卡顿。监测到卡顿后，获取当前堆栈信息上报（BSBacktraceLogger）

### 卡顿优化
* 尽量使用不使用透明
* 像素对齐
*  避免离屏渲染，造成离屏渲染可能的做法：

![卡顿优化](https://i.loli.net/2019/07/12/5d28a2378bd6521681.png)

### ASDK 的核心
* 尽可能的将主线程的任务挪到异步线程
* 异步任务回调主线程是模型 Core Animation，在 `runloop` 注册 `BeforeWaiting` 或者 `Exit` 事件，优先级比 Core Animation 更低。收到事件后执行提交任务。

![ASDK 的核心](https://i.loli.net/2019/07/12/5d28a27df1e6888480.png)
