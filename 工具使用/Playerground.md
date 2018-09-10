## Playground
在这篇文章中将记录我在使用 `Playground` 中遇到的问题。

### 让 Playground 具备多线程调试功能
记得之前用 Playgound 做了一些关于 UI 方面的练习，涉及到了回归主线程的操作，但是一直没有效果，加了延时代码也不行，遂作罢。今天得知原来是需要在 `Playground` 中打开它的“延时运行”特性，代码如下：
```Swift
import PlaygroundSupport
PlaygroundPage.current.needsIndefiniteExecution = true
```
### 使用 Playground 做 UI 测试
首先需要导入 `PlaygroundSupport` 库，然后正常写 UI 代码，最后给 `PlaygroundPage.current.liveView` 赋值上我们需要展示出来的 view 即可，如下所示：
```Swift
import UIKit
import PlaygroundSupport

class ViewController: UITableViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
    }
}

extension ViewController {
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 10
    }
    
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = UITableViewCell()
        cell.textLabel?.text = "\(Int(arc4random_uniform(100)))"
        return cell
    }
}


PlaygroundPage.current.liveView = ViewController()
```

当然，如果这样直接运行是肯定不能看到 UI 测试界面的，需要 `View` -> `Assistant Editor` -> `show Assistant Editor` 。


