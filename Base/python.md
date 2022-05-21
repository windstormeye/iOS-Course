## 模块
每个 `.py` 文件的主文件就是模块名称，想要导入模块时必须使用 `import` 关键词来制定模块名称。若要引用模块中定义的名称，则必须在名称前加上模块名称与一个 “`.`" 符号。

## __pycache__
CPython 将 .py 文件编译后的字节码文件，如果之后再次导入同一个模块，检测到源码文件没有更改过，就不会再对源码重头开始进行语义和语法解析等操作。

## 获取命令行参数
可通过 `sys` 模块中的 `argv` 列表，argv[0] 保存的是源码的文件名：

```python
import sys
print('hello', sys.argv[1])
```

## 包
文件夹中一定要有一个 `__init__.py` 文件，该文件夹才会被视为一个软件包，并且软件包名称会成为命名空间的一部分。

## python 中所有数据都是对象

## python3 之后，整数类型为 `int`，不再区分 `int` 和 `long` 

## type()
想知道某个数据的类型可以使用 `type()` 函数：
```python
>>>type(10)
<class 'int'>
```

## python 支持复数的直接表示法
```python
>>> a = 3 + 2j
>>> b = 5 + 3j
>>> a + b
(8 + 5j)
```

## 格式化字符串
### 旧方式
```python
>>> 'hello, %s!' % 'world'
```

### 新方式
```python
>>> 'hello, {}!'.format('world')
```

## 字典
当从字典中通过某个 key 取值时，当 key 不存在，则会报 `KeyError` 的错误，此时可以通过 `in` 来判断：

```python
>>> nums = {'aha': 2333}
>>> 'wow' in nums
False
>>> nums['wow']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'wow'
```

## 需要处理小数且还需要精确的结果
`import decimal`，其它细节查资料

## 指数运算
```python
>>> 2 ** 10
1024
```

## 比较与赋值
`== != < >` 等符号用来比较值，`is, is not` 用来比较两个对象的引用是否相等

## * 与 **
### `*`
如果事先无法预期要传入的自变量个数，可以在定义函数的参数时使用 `*`，表示该参数接受不定长度的自变量：

```python
def sum(*numbers):
    total = 0
    for number in numbers:
        total += number
    return total
```

### `**`
让指定的关键词自变量收集为一个字典：

```python
def ahaha(**user_settings):
    methon = user_settings.get('methon', 'GET')
    contents = user_settings.get('contents', '')
```