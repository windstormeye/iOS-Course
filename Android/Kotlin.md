# Kotlin 问题汇总

## 语法
### `var` 和 `val` 的区别
`var` 与我们之前见到的 `var` 概念一致，但 `val` 取代了以往 `let` 作用。kotlin 中 `let` 另有他用。
```kotlin
var a: String = "initial"  // 1
println(a)
val b: Int = 1             // 2
b = 2
// 报错：Val cannot be reassigned
```

### `vararg`
本质上是个 `Array<String>` 的语法糖，但可以用“逗号”分隔开参数。在保证参数类型一致的情况下可以这么传参：
```kotlin
class MutableStack<E>(vararg items: E) {              // 1

  private val elements = items.toMutableList()

  fun push(element: E) = elements.add(element)        // 2

  fun peek(): E = elements.last()                     // 3

  fun pop(): E = elements.removeAt(elements.size - 1)

  fun isEmpty() = elements.isEmpty()

  fun size() = elements.size

  override fun toString() = "MutableStack(${elements.joinToString()})"
}

fun <E> mutableStackOf(vararg elements: E) = MutableStack(*elements)

fun main() {
  val stack = mutableStackOf(0.62, 3.14, 2.7)
  println(stack)
}
```

### class

```kotlin
// 可以声明一个没有任何属性的 class，kotlin 会自动创建一个无参构造函数
class Customer
```

* kotlin 的类默认是 `final`，如果该类想要被继承，需要用 `open` 进行修饰；
* kotlin 的方法默认同样也是 `final`，在类中，如果想要的该方法可以被重载，需要用 `open` 进行修饰，并且在重载方法前加上 `override` 修饰；
  ```kotlin
  open class Dog {                // 1
    open fun sayHello() {       // 2
        println("wow wow!")
    }
  }

  class Yorkshire : Dog() {       // 3
      override fun sayHello() {   // 4
          println("wif wif!")
      }
  }

  fun main() {
      val dog: Dog = Yorkshire()
      dog.sayHello()
  }
  ```
#### 继承有参构造类
不得不说，这种写法真是太奇特了。

```kotlin
open class Tiger(val origin: String) {
    fun sayHello() {
        println("A tiger from $origin says: grrhhh!")
    }
}

class SiberianTiger : Tiger("Siberia")                  // 1

fun main() {
    val tiger: Tiger = SiberianTiger()
    tiger.sayHello()
}
```