---
title: macOS开发（词法分析器）
date: 2018-04-16 10:58:46
tags:
- macOS
- Swift
- 编译原理
---

这是我的第一个存粹基于Swift的project，重点是用Swift进行macOS的开发，而不是词法分析器。

对于开发桌面应用来说，目前可供参考的框有windows原生“古老的”MFC、常规的跨平台Qt、目前最火的跨平台[electron](https://github.com/electron/electron)，以及macOS原生的Cocoa。

之前已经尝试过了使用Qt进行构建桌面程序，做了好几个小东西后，越来越发现Qt的强大，不过目前从Qt官方透露出来的消息可以看出，目前他们正在对嵌入式开发，也就是车载系统的开发下大力气，使用Qt类库即可完成车载系统应用的开发，这确实是个更加值得玩味的事情。在做出最终使用Swift进行macOS开发前，一直在纠结到底是学习使用跨平台框架进行开发还是直接走原生，最后思来想去，一直缺少使用Swift进行开发的机会，可以借助这个机会切入。因此，

**在保证编译原理的实验的要求下，使用了Swift进行了原生的macOS开发。（只能作为参考）**

首先来看效果完成图，

<img src="https://i.loli.net/2018/04/16/5ad42235469ef.gif" width = "90%" height = "90%" align=center />

**改进版本一**
上一版本的讲解没有加入字符表，真是尴尬，这是加好字符表的完成图，

<img src="https://i.loli.net/2018/04/16/5ad497098ea39.gif" width = "90%" height = "90%" align=center />

### 概念明晰

毕竟本质上是macOS的开发，所以此篇文章会稍微往开发方面侧重一些。在macOS上来说，其使用的`Cocoa`框架进行GUI程序的搭建，而不是iOS中的`cocoa Touch`框架，两者的差别不只是说换个名字这么简单，其中最大的差别在iOS开发中一般来说，我们都是只使用一个`window`，使用`viewController`进行相关页面的跳转，而在macOS的开发中，一个App可以有理论上无数多个`window`（一般没这么无聊的需求），而且还有`windowController`的概念，以及`keyWindow`和`mainWindow`的区别，坐标系换了，甚至连各大基础组件的继承父类也换了。

而且在macOS开发上开始更加推荐使用SB进行布局，而不用再去手撸布局了，就单一个`NSTableView`，如果我们还是像iOS开发中的那样直接在SB中按住控件拖拽入相关.m文件中，你会发现拖拽进来的居然是个`NSScrollView`， 而正确的想要拖拽进一个`NSTableView`，你要做的是需要在SB已拖拽控件详情列表选择对应的`View Controller Scene`，然后在其中选择对应的`NSScrollView` -> `NSClipView` -> `NSTableView`，这样才能正确的选择出对应的`NSTableView`。

这算是一个比较大的区别吧，因为在iOS中`UITableView`继承于`UIScrollView`，其又继承于`UIView`，但是在macOS中却多出了一个`UIClipView`。其实还有个比较大的区别，对应`NSTableView`的使用，同样也是有`delegate`和`dataSource`两个代理对象，但是其中的相关代理方法和iOS却是大不相同，甚至可以说是非常的奇妙了。🙂

 关于这个项目的怎么布局就不说啦，因为已经关掉了放大缩小功能，可以先暂时不用考虑布局相关内容，所以给大家放张图即可。再强调一次，在iOS开发中我们有时会拒绝使用的SB或者其它布局方式，但是在macOS开发中，说句良心话，“真的，去使用自动布局吧😂”

 <img src="https://i.loli.net/2018/04/16/5ad4296169973.png" width = "100%" height = "100%" align=center />


### 实验相关

实验要求完成一个词法分析器，看了实验指导书上的相关要求，莫名感觉老师应该挖了坑🙂，给了一个非常简单的样例输入和输出，如下：

```
可以参考下面的示例:
输入字符串if  i>=15 then  x := y;
输出：
（3，‘if’）
（1，0）	 // i的符号表入口为0
（4，‘>=’）
（2，‘15’）
（3，‘then’）
（1，1） // x的符号表的入口为1
（4，‘：=’）
（1，2)   // y的符号表的入口为2
（5，‘；’）
```

目前来说，我做到现在的词法分析器能够分析出如下所示的C语言代码：
```c
#include "stdio.h"
int main() {
    /*这是注释*/
    int a = 0, b = 0, c = 0;
    printf("%d %d %d", &a, &b, &c);
    return 0;
}

// int a = 0, b = 1;
```

词法分析器的关键就在于这个词法分析上，词法分析接受源码的输入，输出token表，而这个token是语法分析阶段的输入。那如何进行词法分析呢？书上讲了一堆的相关方法，梳理了一遍，我觉得逻辑应该是这样的，首先拿到对应的正则表达式，一般来说就是你要进行分析的对应编程语言的正则表达式集，接着对正则表达式集做出NFA（不确定的有穷自动机）状态图，一般来说到NFA这就行了，但这是对我们正常人来说的，对于人来说看NFA是最符合逻辑的，但是对于计算机来说还差点意思，我们需要把所以的相关逻辑，何时进何时退等逻辑都告诉它，因此最后一步为把NFA状态图转为DFA（确定的的有穷自动机）状态图，DFA则相当于NFA来说满足了计算机的正常处理逻辑。

emmm。以上都是官话，现在来看真实的样子。🙂。根据之前分析的结果，我们需要接收一个由客户端界面上传递而来的字符串，当然，这个字符串是带有`\t \n`等特殊字符的，我们也要对其做出判断。需要一个字典数组，用来保存token对，需要多个特殊字符数组，用于判断当前接收到的字符串所属类型，为了判断的方便，我把这坨特殊字符做了八大归类，分别为操作符、界符、注释、非法字符、输入输出字符、关键字、正常字符、头文件。

这个块有个需要注意的地方，在Swift中，如果你对成员变量（属性）在声明的时候没做初始化，那么在init这个对象的时候会给你报错，告诉你缺少实例化。我们可以这么解决，第一种：在对应的类中成员变量声明时，先显式的说明初始值；第二种：设置成员变量为可选类型，Swift中的可选类型包括nil嘛，嘿嘿嘿；第三种：在对应类的init方法中初始化对应的成员变量即可。我采用了第一种方法。

```swift
public var inputCodeString: String = ""
public var token = Array<[String : String]>()

// 操作符 OPT
private let operatorWords = ["+", "-", "*", "/", "%", "<", ">", "=", ">=", "<=", "==", "!="]
// 界符 MAR
private let marginalWords = [";", "{", "}", "(", ")", " ", "\r", "\t", "\"", ",", "&"]
// 注释 ANT
private let annotatedWords = ["//", "/*", "*/"]
// 非法字符 ILG
private let illegalWords = [".", "?", "!", "@", "#", "$", "^", "~", ":"]
// 输入输出 INO
private let intoutWords = ["%d", "%c", "%f", "%lf"]
// 关键字 KEY
private let keyWords = ["break", "case", "char", "const", "printf",
                        "continue", "default", "double", "else",
                        "enum", "extern", "float", "for", "goto",
                        "if", "int", "long", "redister", "return",
                        "sizeof", "static", "struct", "while",
                        "switch", "typedef", "unsigned", "void", "#include"]
// 正常字符 NOR
// 头文件 HED
```

而我们的init方法如下所示，接收一个字符串，赋值给对应的成员变量即可。如果你想连init方法都不写也行， 用对应的setter方法即可，我非常推荐这种写法，Swift的setter/getter方法的写法真的是太美妙了😂。
```swift
// MARK: init方法
// designer
public init(inputCodeString : String) {
    self.inputCodeString = inputCodeString
}
```

接下来是我们的类型判断方法，丢入一个切割好的String，返回一个对应的String类型，其中在判断是否为正常字符，即数字、大小写字母用到了Swift中的正则写法，也被称为“谓词匹配”。

```swift
private func contanierType(keyString: String) -> String {
    if operatorWords.contains(keyString) {
        return "操作符"
    } else if marginalWords.contains(keyString) {
        return "界符"
    } else if annotatedWords.contains(keyString) {
        return "注释"
    } else if keyWords.contains(keyString) {
        return "关键词"
    } else if illegalWords.contains(keyString) {
        return "非法字符"
    } else if inputNumberOrLetters(keyString) {
        return "正常字符"
    } else if intoutWords.contains(keyString) {
        return "输入输出"
    } else {
        return ""
    }
}

private func inputNumberOrLetters(_ name: String) -> Bool {
    if name.isEmpty {
        return false
    }
    let regex = "[a-zA-Z0-9]*"
    let predicate = NSPredicate(format: "SELF MATCHES %@", regex)
    let inputString = predicate.evaluate(with: name)
    return inputString
}
```

最后就是我们的词法分析核心方法了，在这个方法中，我们需要先进行字符串的切割，而这个切割的标准就是从一个空格、一行的开始或者`\t`tab键的开始，结束为空格或者`\n`回车字符，所以，当每次进行扫描时遇到空格即为当前追加字符的开始或者结束，同时也可以说是遇到了界符，即可结束。我把注释`\\`也做为了界符，因为当扫描到注释符号时，实际上后边的内容我们都不需要关心，直接`continue`跳过即可。

关于字符表的插入，主要的实现逻辑有，当检测到当前的缓冲字符串为正常字符时，开启正常字符串BOOL，在检测下一个字符时，若为操作符，则开启操作符BOOL，当接收的第三个缓冲字符串还为正常字符串时，且正常字符串BOOL和操作符BOOL同时为真，即可进行插入字符表，并且进行三次删除操作，把之前的操作符、正常字符、关键词三个都删掉。

同时也做了头文件的判断，判断头文件就只需要坚持出当前扫描到的字符串中是否带有`.h`即可😂。需要稍微注意的是，每次执行完一次扫描字符串的类型判断，记得把扫描字符串清空即可，最后把整个token返回出去。

```swift
// MARK: 词法分析方法
public func lexicalAnalysis() -> Array<[String : String]> {
    var tampString = ""
    // 注释检测符号
    var annotatedStatus = false
    var normalWordStatus = false
    var operatorStatus = false

    for singleChar in inputCodeString {
        if singleChar == "\n" {
            annotatedStatus = false
            continue
        }

        // 检测界符
        if !marginalWords.contains(String(singleChar)) {
            tampString.append(singleChar)
        } else {
            // 检测注释
            if !annotatedStatus {
                if annotatedWords.contains(tampString) {
                    annotatedStatus = true
                }

                if tampString.contains(".h") {
                    token.append(["头文件": tampString])
                }

                let tokenString = contanierType(keyString: tampString)
                if !tokenString.isEmpty {

                    // 插入字符表表相关逻辑
                    if tokenString == "正常字符" {
                        normalWordStatus = true
                    }
                    if normalWordStatus {
                        if tokenString == "操作符" {
                            if tampString == "=" {
                                operatorStatus = true
                                normalWordStatus = false
                            }
                        }
                    }
                    if operatorStatus {
                        if tokenString == "正常字符" {
                            workTable.append(tampString)
                            operatorStatus = false
                            normalWordStatus = false
                            token.append(["字符表": String(workTable.count - 1)])
                            if (token[token.count - 2]["操作符"] != nil) {
                                token.remove(at: token.count - 2)
                            }
                            if (token[token.count - 2]["正常字符"] != nil) {
                                token.remove(at: token.count - 2)
                            }
                            if (token[token.count - 2]["关键词"] != nil) {
                                token.remove(at: token.count - 2)
                            }
                            // 填充到字符表中后需要把当前缓存字符串清空
                            tampString = ""
                            continue
                        }
                    }

                    // 除了注释都可以被加入
                    if !annotatedStatus {
                        token.append([tokenString: tampString])
                    }
                }

                if ![" ", "\n", ""].contains(String(singleChar)) {
                    token.append(["界符" : String(singleChar)])
                }

                tampString = ""
            }
        }
    }

    return token
}
```

### macOS开发部分

进入到了macOS开发部分。在该部分中，我们主要完成的事情后：
1. 界面搭建；
2. 数据获取；
3. 界面刷新。

界面搭建大家就用SB吧，对自己狠一些🙂。刚开始我也是十分的抵制SB，布局什么的不经过自己的手总感觉缺少了点什么。

在数据获取这块，我们首先需要对输入做一个判断，该词法分析器提供“文件输入”和“文本输入”两种输入方式，对于“文本输入”，直接就从`NSTextField`中获取即可，找了半天，不是`.text`也不是`.textString`，试了很多属性都不行，翻了`NSTextField`的头文件，也没写明白到底通过哪个属性获取，最后发现，居然是`.stringValue`。🙄。真是无语。

```swift
@IBAction func lexicalAnyButton(_ sender: Any) {
    let analysisTool = PJAnalysisTool.init(inputCodeString: inputTextField.stringValue)
    tokenArray = analysisTool.lexicalAnalysis()
    outputTableView.reloadData()
}
```

而通过“文件输入”，过程有些曲折，使用了`NSOpenPanel`类，苹果官方开发文档对其有这么个介绍`The Open panel for the Cocoa user interface.`，打开一个为Cocoa用户界面面板（就是文档选择器啦），我对该面板做了如下设置，能选择文件，不能选择文件夹，不允许多选。

当我们按下面板上的`OK`或者`确定`按钮，通过Data（相当于NSDate）类载入文件内容，将该文件内容转码为`utf-8`编码，拿到最终的编码字符串后，丢入词法分析类，返回一个字典数组，而该字典数组就是我们`NSTableView`进行数据刷新的数据源。

```Swift
@IBAction func selectFile(_ sender: Any) {
    let panel = NSOpenPanel.init()
    panel.canChooseFiles = true
    panel.canChooseDirectories = false
    panel.allowsMultipleSelection = false

    let finded : Int = panel.runModal().rawValue
    if finded == NSApplication.ModalResponse.OK.rawValue {
        for url in panel.urls {
            let codeData = try? Data.init(contentsOf: url)
            let codeString = String(data: codeData!, encoding: String.Encoding.utf8)
            print(codeString!)
            let analysisTool = PJAnalysisTool.init(inputCodeString: codeString!)
            tokenArray = analysisTool.lexicalAnalysis()

            outputTableView.reloadData()
        }
    }
}
```

接下来就到了`NSTableView`的相关代理方法实现了，`NSTableView`有两个必须实现的代理方法`numberOfRows`和`objectValueFor`，其中`objectValueFor`可以给空值。

剩下的还有一个值得注意的地方，目前我只找到了对`NSTableView`进行分栏通过`identifier`标识符去做，也就是说，一个分栏对应一个重用符，应该还会有其它的设置方法。

```swift

    func numberOfRows(in tableView: NSTableView) -> Int {
        return tokenArray.count
    }

    func tableView(_ tableView: NSTableView, objectValueFor tableColumn: NSTableColumn?, row: Int) -> Any? {
        return nil
    }

    func tableView(_ tableView: NSTableView, heightOfRow row: Int) -> CGFloat {
        return 20
    }

    func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {

        let idString = tableColumn?.identifier

        if idString!.rawValue == "AutomaticTableColumnIdentifier.0" {
            var cellView = tableView.makeView(withIdentifier: idString!, owner: self)
            if (cellView != nil) {
                cellView = NSTableCellView.init(frame: NSMakeRect(0, 0, (tableColumn?.width)!, 20))
            } else {
                for view in (cellView?.subviews)! {
                    view.removeFromSuperview()
                }
            }

            let textField = NSTextField.init(frame: NSMakeRect(0, 0, (tableColumn?.width)!, (cellView?.frame.size.height)!))

            textField.stringValue = Array(tokenArray[row].keys)[0]
            textField.isBordered = false
            textField.isEditable = false
            textField.alignment = .left
            textField.backgroundColor = NSColor.clear
            cellView?.addSubview(textField)

            return cellView
        } else {
            var cellView = tableView.makeView(withIdentifier: idString!, owner: self)
            if (cellView != nil) {
                cellView = NSTableCellView.init(frame: NSMakeRect(0, 0, (tableColumn?.width)!, 20))
            } else {
                for view in (cellView?.subviews)! {
                    view.removeFromSuperview()
                }
            }

            let textField = NSTextField.init(frame: NSMakeRect(0, 0, (tableColumn?.width)!, (cellView?.frame.size.height)!))
            textField.stringValue = Array(tokenArray[row].values)[0]
            textField.isBordered = false
            textField.isEditable = false
            textField.alignment = .left
            textField.backgroundColor = NSColor.clear
            cellView?.addSubview(textField)

            return cellView
        }
    }
```

好了。以上就是我使用Swift完成的第一个小实验，如果在课程持续进行的过程中发现了有不足，会持续更新，目前完成的词法分析器的识别功能还是太简单。后续我会尽量把做的新东西都往Swift上迁，慢慢的把Objective-C的地位给抹掉，毕竟它现在的地位在国内还是太硬了。

相关demo见：[词法分析器](https://github.com/windstormeye/MacMorePractices/tree/master/lexicalAnalysisTool)
