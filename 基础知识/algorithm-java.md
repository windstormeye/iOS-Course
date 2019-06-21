1. `public static void`和`public void`的区别：`public static void`所标明的静态方法能够用类名直接调用，但是静态方法不能够调用非静态方法。

2. final关键字浅析：[https://www.cnblogs.com/dolphin0520/p/3736238.html](https://www.cnblogs.com/dolphin0520/p/3736238.html)

3. `const`和`static`关键词：
* [http://blog.sina.com.cn/s/blog_668aae780101m4ex.html](http://blog.sina.com.cn/s/blog_668aae780101m4ex.html)
* [https://www.jianshu.com/p/1598004e8215](https://www.jianshu.com/p/1598004e8215)

`const` 修饰为只读常量，只能初始化一次，`static` 修饰的变量和函数只能在当前模块或文件中可见。

4. 如何才能将一个`double`变量初始化为无穷大？使用java内置常数，`Double.POSITIVE_INFINITY`和`Double.NEGATIVE_INFINITY`

5. 能够将`double`类型的值和`int`类型的值相互比较吗？不通过类型转换不行！但是java一般会自动进行所需的类型转换，例如，如果`int x = 3`，则`(x < 3.1)`为`true`，java会在比较前将x转换为`double`类型（因为3.1为`double`类型的字面量）

6. java二维数组表示：`int[][] arr = { {1,2,3}, {4,5,6} };`
