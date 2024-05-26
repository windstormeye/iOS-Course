## Swift 调试

## lldb
OC 代码中可以很方便的直接通过发消息的方式调用私有方法，但 Swift 中却因为静态语言的原因无法直接通过类似 OC 中发消息的方式调用私有方法，但 apple 对 Swift 保留了通过 `perform` 的方式，可以直接通过传入私有方法字符串的方式调用。

```shell
(lldb) po pipController.perform("_ivarDescription")
```

```oc
[foo _ivarDescription]; // 列举了所有的实例变量、类型和值。
[foo _methodDescription]; // 大致和_shortMethodDesctiption一样，但是更加详细还包含了超类的方法；
[foo _shortMethodDescription]; // 列举了所有实例和类方法；
```