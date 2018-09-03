## Playground
在这篇文章中将记录我在使用 `Playground` 中遇到的问题。

### 让 Playground 具备多线程调试功能
记得之前用 Playgound 做了一些关于 UI 方面的练习，涉及到了回归主线程的操作，但是一直没有效果，加了延时代码也不行，遂作罢。今天得知原来是需要在 `Playground` 中打开它的“延时运行”特性，代码如下：
```Swift
import PlaygroundSupport
PlaygroundPage.current.needsIndefiniteExecution = true
```

