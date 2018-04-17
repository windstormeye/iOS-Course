---
title: ONEUIKit/ONEProgressHUD
date: 2018-04-09 21:29:24
tags:
- iOS
- HUD
---

## ONEUIKit/ONEProgressHUD
ONEUIKit/ONEProgressHUD为当前滴滴出行App中的HUD，我们先来看看其的样式，

<img src="https://i.loli.net/2018/04/09/5acb3b03dbe77.png" width = "40%" height = "40%" align=center />


图中的“网络异常，请稍后再试”的整个HUD样式，就是ONEUIKit下的ONEProgressHUD，这次就来一次仔细看看它到底是怎么一回事。

跟`ONEProgressHUD`同样的类库还有`SVProgressHUD`和`MBProgressHUD`，ONEProgressHUD同时也受到了MBProgressHUD的影响，参考了其部分实现。滴滴数据App本身之前用的SVProgressHUD，这个HUD库可以说是目前iOS开发最火的第三方HUD库了，整理了一下ONEProgressHUD的实现流程，如下所示，

<img src="https://i.loli.net/2018/04/09/5acb3d8c3c03a.jpeg" width = "100%" height = "100%" align=center />



因为并不能贴任何有关该库代码（内部库），所以大家就看我总结的图吧，图中展开了一个secondary方法，并且按照该方法的调用栈进一步展开，因为并不能贴代码，只能讲一些大概的核心方法，大家明白这么个意思就行，再说一句，请期待我们的`ifLabKit`喔~😜
