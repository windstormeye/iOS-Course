## C++ 拾遗

## 为什么 main 函数要 return 0？
在大多数系统中，main 的返回值被用来指示状态。返回值 0 表明成功，非 0 的返回值的含义由系统定义，通常用来指出错误类型。

## do {...} while(0)
可能在一个方法中会出现 ABCD 四段代码块，其中 BC 两块有虚拟的包含关系，B 执行不了 C 也不执行，直接到 D。不用 do-while 也能解，就是不好看罢了。

## 带符号类型和无符号类型
带符号类型可以表示正数、负数或 0，无符号类型仅能表示大于 0 的值。

## 创建一个 C++ 对象何时用 new 何时不用？
一般来说，通过 `new` 关键词创建出的对象可以超出当前逻辑作用域，除非手动 delete 释放，否则不会自动释放。而通过类型 `Object obj;` 方式初始化的对象其生命周期终止与当前作用域。

## 顶层 const 和底层 const 之分
`const int i` 为顶层 const，i 的值无法被改变。

`int* const i` 为顶层 const，i 的值也无法改变，但可以修改指针所只指向的地址。

`const int* i` 为底层 const，i 的值可以改变，但无法修改指针所指向的地址。

综上，const 最近修饰的内容不可变。

## C++11 里的 const 和 constexpr
`const` 只作为“只读”。
`constexpr` 只作为“常量”。

## std::string size()
size 函数返回值的类型是 `size_type`，从 C++ Primer 书中看到的推测是一个无符号类型的值，因此绝对不可以与一个有符号且可能未负数的值进行比较，否则会出现明明比它大，却比它小的情况。

```c++
int a = -1;
std::string b = "123";
if (b.size() < a) {
    std::cout << "woc?"; // 会打印出 woc?
}
```

因此，当我们需要定义一个变量作为遍历或取 `std::string` 类型里的值时，可以把该变量类型定义为 `std::size_type` 类型，可以保证肯定不会出现小于 0 的场景。

## std::string 相加
当两个变量其中一个明确为 `std::string` 类型时，可以通过 + 运算符进行相加操作，若两个变量都通过字面量的方式进行相加，则是非法的，因为历史原因，也为了和 C 兼容，C++ 里的字符串字面量并不是标准库 `std::string` 类型。  

## std::vector
是模板而不是类型，其为 C++ 的“类模板”。

所有使用了迭代器的循环体，都不要向迭代器所属的容器增删元素。

## 在类的成员函数后加 const
当我们明确外部调用某些成员函数不可修改类内成员变量的内容时，可以通过在对应的函数声明后添加 const 关键字来告诉编译器，该方法不允许修改任何类内成员变量。

## class 和 struct 的区别
只有一个，默认的访问权限。如果我们明确了定义类的所有成员都是 public 的，则可以使用 struct。

## 如何声明一个使用默认构造函数初始化的对象？
```c++
Object obj; // 正确
Object obj(); // 错误，初始化了一个函数
```

## 类的静态成员变量
当一个变量只存在这个类内，但该类又会创建出多份，且都会使用同一个该变量，这种情况下可以使用静态成员变量。

## 迭代器的删除
对于关联容器，删除当前的 iterator，仅仅会使当前的 iterator 失效，只要在调用 erase 时，递增当前的 iterator 即可。这是因为 map 之类的容器，使用了红黑树来实现，插入、删除一个结点不会对其他结点造成影响。

对于顺序容器，删除当前的 iterator 会使后面所有元素的 iterator 都失效。这是因为 vector 、deque 使用了连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。不过 erase 方法可以返回下一个有效的 iterator 。

## std::string_view
C++17 引入`std::string_view` 来优化string的性能。`string_view` 本身不own内存，它只维护了一个指针和长度。而 `string_view` 仅存储了对原始数据的引用，该过程不涉及任何复制操作，所以它在处理大字符串时，性能上要比 string 更优。

```c++
#include <string_view>
static const string_view str = "HELLO ByteDance!"; // 没有任何开销
char foo() {
    return str[2];
}
等价
#include <string_view>
char foo() {
    return 'L';
}
```

使用`string_view`的注意事项：
- `std::string_view` 可以与`std::string`互相转换，但要注意`string_view`的生命周期问题。由于`std::string_view`并不持有字符串的内存，所以它的生命周期一定要比源字符串的生命周期长，源字符串被消毁，行为未定义。
```C++
std::string_view PrintStringView() {
    std::string s = "How are you..";
    std::string_view str_view = s;
    return str_view;
}
// 行为未定义，悬垂指针
std::cout << "PrintLocalStringView: " << PrintStringView() << std::endl;
```

- `std::string_view`并不提供修改其引用的字符串的方法。任何尝试修改string_view引用的字符串的操作都可能导致未定义的行为。

- 与其他字符串类型不同，应该按值传递`string_view`，就像传递int或double一样，因为`string_view`是一个小值。

- `string_view`不一定以null结尾。因此，使用printf函数输出`string_view`是不安全的:
```c++
printf("%s\n", sv.data()); // DON’T DO THIS
// 可以像string或const char*使用<<输出string_view：
std::cout << "Took '" << s << "'";
```


## 尽量提前使用reserve/resize来避免不必要的内存分配
对于vector和string，增长过程是这样来实现的：每当需要更多空间时，就调用与realloc类似的操作。这一类似于realloc的操作分为四个步聚：
1. 分配一块大小为当前容量的某个倍数的新内存。在大多数实现中，vector和string的容量每次以2的倍数增长，即，每当容器需要扩张时，它们的容量即加倍。
2. 把容器的所有元素从旧的内存拷贝到新的内存中。
3. 析构掉旧内存中的对象。
4. 释放旧内存。

```C++
// Bad Code !!!
std::vector<int> container;
for (int i = 0; i < 1000; ++i)
{
    container.push_back(i);
}

改为:

std::vector<int> container;
container.reserve(1000);
for (int i = 0; i < 1000; ++i)
{
    container.push_back(i);
}
```

## 容器元素
元素拷贝：当向容器中加入对象时，存入容器的是你所指定的对象的拷贝。当从容器中取出一个对象时，你所得到的是容器中所保存的对象的拷贝。进去的是拷贝，出来的也是拷贝(copy in, copy out)。
所以如果容器存储大对象，一般都会存指针以提升性能（指针的拷贝比对象拷贝代价少）

元素析构：STL的容器自身被析构时，它们会自动析构容器内所包含的每个对象。如果容器内的元素是通过new创建的，那么在释放容器的时候，指针的“析构函数”不会做任何事情，因此new所创建的内存就不会释放掉。如果不手动delete掉，那么就会造成内存泄漏。

为了防止内存泄漏，最简单的方法就是通过delete释放防止内存泄漏
但是这种方式仍存在一个问题：如果在new操作和delete操作之间程序抛出了异常导致程序终止，那么delete语句将永远不会执行，同样也会产生内存泄漏。

使用智能指针来保证内存不会泄露，特别是map的value如果存储大对象，使用智能指针可提升性能，指针的拷贝比对象拷贝代价少，因此建议存储为map<key, std::shard_ptr<value>>，这样既能保证指针高性能，又不需要担心erase的析构的问题。

与析构类似，remove操作也需要注意：当容器中存放的是new分配对象的指针时，应该避免使用remove和remove_if。如果容器中存放的不是普通指针，而是具有引用计数功能的智能指针，那么就可以直接使用erase-remove的习惯用法。

