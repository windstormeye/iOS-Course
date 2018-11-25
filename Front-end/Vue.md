# Vue

## MVVM 数据绑定
在学习 `Vue` 的过程中，非常好奇 `Vue` 自身的数据绑定模式是否与我之前在 iOS 上接触到的数据绑定认识一致，然后又在资料中发现了 `Object.defineProperty()` 和 `Object.defineProperties` 方法，相当于对一个属性进行了 `setter` 和 `getter` 的监听，然后根据监听结果重新更新元素，大致如下所示：

```html
<html>
  <head>
  </head>
  <body>
    <span id="span"></span>
    <script>
      var obj = {
        pwd: "123123"
      };
      Object.defineProperty(obj, "userName", {
        get: function () {
          console.log("get init")
        },
        set: function (newValue) {
          console.log("set init")
          document.getElementById("span").innerText = newValue
        }
      });
    </script>
  </body>
</html>
```

看到这段代码后，瞬间就大彻大悟！其实也不是什么特别高深的内容（当然还是有其它值得学习的地方～），而且这么做耦合度十分的高，就目前的学习内容来看，`Vue` 中对 **组件化** 的核心思路跟之前在 iOS 中流程还是一致的～