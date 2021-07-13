---
title: Ruby 基础
date: 2021-02-13 10:05:22
tags:
  - 入门
  - Ruby
categories:
  - [全栈, Ruby]
---

Ruby 的介绍与基础。

<!-- more -->

# 基本

## 运行 Ruby 的方式

- `irb` 可交互式命令行
- `ruby` + 文件路径

## 变量

- 没有 `var` `const` `let` 等关键字
- 局部变量：小写字母开头或 `_` 开头
- 全局变量：`$` 开头
- 类变量：`@@` 开头
- 实例变量：`@` 开头

## 常量

- 全大写，且默认为全局，比如 `RUBY_VERSION`

## 多重赋值

- `a,b,c = 1,2,3`
- `a,b = [1,2]`
- `a,b,*c = 1,2,3,4,5`
- `a,*b,c = 1,2,3,4,5`
- `a,b = b,a`

## 字符串

- 单引号，没有转义，写 `'\n'` 里面就是 `\n`
- 双引号，有转义
- 多行字符串

```ruby
s = <<eos
dfa123
fad
eos
```

## log

- `print`
- `puts` put string
- `p`

## 注释

- 单行注释 `#`
- 多行注释

```ruby
=begin
注释
=end
```

# 控制语句

## if

```ruby
if a > 3 then p '大' end
if a > 3 then p '大' else p '小' end
p(if a > 3 then '大' else '小' end)
p(if a > 4
  '大'
  elseif a > 2
  '中'
  else
  '小'
  end)
```

```ruby
return if error
return if not success
return unless success
```

## case

类似 `switch-case` 单不需要 `break`

```ruby
x = case a
when 1
  '1'
when 2
  '2'
else
  'others'
end
```

# 循环语句

## times

```ruby
7.times do
  p '哈哈'
end

7.times { p '哈哈' }
```

`times` 是函数，`do end` 和 `{}` 是代码块，作为 `times` 函数的最后的参数，一般多行用 `do end`，单行用 `{}`

```ruby
7.times do |i| p "#{i} time" end
```

`|i|` 相当于给你的参数，表示第几次

## each

```ruby
[1, 2, 3].each do |item| p item end
```

```ruby
(1..7).each do |n| p n end
```

## for

```ruby
for i in 1..7 do p i end
```

## while/until

```ruby
i = 1
while 1 < 3
  p i
  i += 1
end

j = 1
until j >= 3
  p j
  j += 3
end
```

## loop

一直循环

```ruby
loop
  p 'ruby'
  if x > 10 then break
end
```

## break continue

`break` 和 `next` 对应 `break` 和 `continue`

# 数据类型

ruby 中只有对象，可以用 `.class` 查看类

- `Integer` `Numeric` 整数
- `Float` 浮点数
- `String` 字符串
- `Array` 数组
- `Regexp` 正则
- `Time` 时间
- `File` 文件
- `Symbol` 符号
- `Exception` 异常
- `Hash` 散列

每个对象都有唯一的 `object_id` 属性

## Array

`arr.methods` 可以看到有什么 api

## Hash

```ruby
me = { name: 'Harvey', age: 20 }
```

上面的 `name` 和 `age` 不是字符串，是 symbol，相当于

```ruby
me = { :name => 'Harvey', :age => 20 }
```

最初的写法相当于上面写法的在 ruby 2 以后的语法糖

symbol 相当于轻量的字符串，功能更少，可以通过 `.to_s` 得到字符串，通过 `.to_sym` 得到 symbol

### 散列的遍历

```ruby
me.each do |key, value| p key, value end
```

### 与 JS 对象的区别

- 不能用 `me.name` 拿到属性，要用 `me[:name]`
  - `'name'` 和 `:name` 是不一样的
- 要给对象声明一个属性为函数，比较麻烦

```ruby
def say_hi
  p 'hi'
end

# 这样写是不行的，他等价于 me[:say_hi] = say_hi()
me[:say_hi] = say_hi
# 要用 lambda 表达式
me[:say_hi] = lambda{ p 'hi' }
me[:say_hi].call
```