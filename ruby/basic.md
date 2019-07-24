# Ruby

## 概念
### 使用 `irb` 进入交互式终端环境

### 单引号和双引号的区别
单引号可以直接把包括的特殊字符进行输出。

### `print`、`puts` 和 `p` 的区别
* 使用 `p` 进行输出，特殊字符不会被转义。

![](https://i.loli.net/2019/07/21/5d34869034b0819432.png)

### 字符串插值
`print "表面积 = #{area}\n"`

### `times` 方法
```ruby
100.times do
  print "All work and no play makes Jack a dull boy.\n"
end
```

### 符号
```ruby
sym = :foo      # 表示符号“:foo”
sym2 = :"foo"   # 意思同上
```

```ruby
> irb --simple-prompt
>> sym = :foo
=> :foo
>> sym.to_s         # 将符号转换为字符串
=> "foo"
>> "foo".to_sym     # 将字符串转换为符号
=> :foo
```

### 散列的创建
```ruby
address = {:name => "高桥", :pinyin => "gaoqiao", :postal => "1234567"}
```
将符号当作键来使用时，程序还可以像下面这么写：
```ruby
address = {name: "高桥", pinyin: "gaoqiao", postal: "1234567"}
```

### 接收输入
```ruby
puts "首个参数: #{ARGV[0]}"
puts "第2 个参数: #{ARGV[1]}"
puts "第3 个参数: #{ARGV[2]}"
```

```shell
> ruby print_argv.rb 1st 2nd 3rd
首个参数: 1st
第 2 个参数: 2nd
第 3 个参数: 3rd
```

### 变量类型
#### 局部变量
以英文字母或者 `_` 开头。
#### 常量
全大写
#### 全局变量
以 `$` 开头。
#### 实例变量
以 `@` 开头。
#### 类变量
以 `@@` 开头

* 变量前加上*，表示 Ruby 会将未分配的值封装为数组赋值给该变量。
  ```ruby
  a, b, *c = 1, 2, 3, 4, 5
  p [a, b, c]    #=> [1, 2, [3, 4, 5]]
  a, * b, c = 1, 2, 3, 4, 5
  p [a, b, c]    #-> [1, [2, 3, 4], 5]
  ```


###  `=~` 符号
The `=~` operator matches the regular expression against a string, and it returns either the offset of the match from the string if it is found, otherwise nil.