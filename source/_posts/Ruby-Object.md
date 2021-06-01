---
title: Ruby 的对象与类
date: 2021-02-13 10:48:11
tags:
  - 入门
categories:
  - [全栈, Ruby]
---

Ruby 的简单的面向对象与类的相关知识。

<!-- more -->

# Class

## Class 可以被重复定义

```ruby
class String
  def to_words
    self.gsub(/[^\da-zA-z\s]/, '') # self. 可以省略
  end
end
```

在重复定义的过程中，相同名字的 Class 会合并。 此处相当于打开 String 类（open class String），并添加 to_words 方法。

## Class 里面可以写任何语句

```ruby
3.times do
  class MyClass
    puts 'Hello'
  end
end
```

在 Class 里面写的语句都会被执行

## 一个例子

```ruby
class MyClass
  # 初始化方法 initialize，在 new 的时候会自动调用
  def initialize
    # 相当于 this.@var
    @var = 1
  end

  def my_method
    @var += 1
    # 会自动 return @var
  end
end

obj = MyClass.new
p obj.class
p obj.my_method
p obj.instance_variables # 打印出所有变量
# obj[:@v] 会报错，因为没有定义 [] 方法
p obj.methods # 打印出所有方法
p obj.methods.grep(/my/) # 打印出包含 my 的方法
p Class.instance_methods.grep(/my/)

# 类实例化之后，实例变量存在对象上，实例方法存在类上
```