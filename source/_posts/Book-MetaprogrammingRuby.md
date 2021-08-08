---
title: 读书笔记：《Ruby 元编程》
date: 2021-07-11 11:49:12
tags:
  - 读书笔记
  - Ruby
categories:
  - [全栈, Ruby]
mathjax: true
---

Ruby 元编程是基于 Ruby2.0 编写的一本老书了，主要学习一下他的思想。

「元编程是编写能在运行时操作语言构件的代码。」——能写代码的代码

<!-- more -->

# 元编程

我们将变量、类、方法等这些称为**语言构件**。对于 C++ 这类语言，运行时就已经看不见他们了，我们无法在运行时询问一个类，他的实例方法有哪些；
但对于 Ruby 等语言，这些语言构件在运行时仍然存活，我们甚至可以在运行时询问某个构件，关于他的相关信息——这被称之为 **内省（introspection）**。

```ruby
class Greeting
  def initialize(text)
    @text = text
  end
  def welcome
    @text
  end
end

my_object = Greeting.new "Hello"
```

我们可以在运行时通过这些方法来询问 `my_object`：

```ruby
my_object.class # => Greeting
my_object.class.instance_methods(false) # => [:welcome]
my_object.instance_variables # => [:@text]
```

我们甚至可以在运行时增加新的实例方法。

从广义上来说，用代码生成器也属于元编程（静态元编程 static metaprogramming），类似于模板；本书讨论的是另一种元编程（动态元编程 dynamic metaprogramming），即编写在运行时操作自身的代码。

# 对象模型 The Object Model

Ruby 中，有很多语言构件：对象、类、模块、实例变量等，元编程操作的就是这些语言构件。所有这些语言构件存在的系统称之为**对象模型**。

## 打开类 Open Classes

```ruby
# 直接修改了 String 这个 class
class String
  def to_alphanumeric
    gsub(/[^\w\s]/, '')
  end
end
```

### 类的定义

1. 类定义中可以放任何语句
2. 类可以重复“定义”，第二次使用 `class` 尝试定义他的时候，他并不会覆盖之前的定义，而是会增加一些内容，这种反复定义，称为 **打开类**

```ruby
3.times do
  class MyClass
    puts "Hello"
  end
end
```  

在很多 Ruby 的库中都使用了打开类来对一些标准的 Ruby 类来进行魔改，比如 `monetize` 包就给 `Numeric` 类增加了 `to_money` 方法

> JavaScript 中其实也可以实现类似的操作，我们可以修改 Array、Number 的 prototype。但是经验上我并不会经常这么做，主要是怕重名或影响到别人的代码。

### 打开类的问题

确实，这种方法容易因为重名覆盖问题，导致其他的副作用。因此，有人称之为 **猴子补丁 Monkeypatch**

## 类的真相

### 对象中有什么

```ruby
class MyClass
  def my_method
    @v = 1
  end
end

obj = MyClass.new
```

1. 实例变量 `[:@v]`：通过 `Object#instance_variables` 方法来查看。对象的类与实例变量没有关系，当给实例变量赋值时，他们就突然出现了。
2. 方法 `[:my_method]`：通过 `Object#methods` 方法查看，他会给出所有继承的方法，我们可以通过 `Array#grep` 方法进行过滤。

> 实例变量存在对象中，而方法存放在类中。

{% note warning %}
需要注意的是，我们不能说 `my_method` 是 `MyClass` 的方法，因为我们不能通过 `MyClass.my_method()` 这样来调用它。

我们只能说 `my_method` 是 `MyClass` 的一个 **实例方法**，因为我们必须要先定义一个 `MyClass` 的实例，然后通过实例调用它。

```ruby
String.instance_methods == "abc".methods  # => true
String.methods == "abc".methods  # => false
```
{% endnote %}

### 类的真相

类本身也是对象，这又有点像 JavaScript 了

```ruby
"hello".class # => String
String.class  # => Class
```

根据前面所述，实例的方法就是类的实例方法。因此，一个类的方法就是 `Class` 的实例方法

```ruby
# false 表示忽略继承的方法
Class.instance_methods(false) # => [:allocate,:new,:superclass]
```

`allocate` 方法是 `new` 方法的支撑方法，`superclass` 则跟继承有关：

```ruby
Array.superclass  # => Object
Object.superclass # => BasicObject
BasicObject.superclass # => nil
```

`Class` 的 `superclass` 是 `Module`，类其实就是带有 `new` `allocate` `superclass` 的增强模块，这三个方法可以让我们按照一定层次创建对象。

- 如果希望把代码 `include` 到别的代码中，就应该使用模块
- 如果希望代码被实例化或继承，应该使用类

### 常量

- 任何以大写字母开头的引用（包括类名和模块名）都是常量
- 我们可以修改常量的值，尽管会得到一个警告
- 常量和变量的最大区别在于作用域不同

```ruby
module MyModule
  MyConstant = 'OuterConstant'
  class MyClass
    # 两个常量有着不同的作用域，他们是不同的
    MyConstant = 'InnerConstant'
  end
end
```

```ruby
class M
  class C
    X = 'a constant'
  end
  C::X   # => 'a constant'
end
M::C::X  # => 'a constant'
```

```ruby
Y = 'a root-level constant'
module M
  Y = 'a constant in M'
  Y     # => 'a constant in M'
  ::Y   # => 'a root-level  constant'
end
```

`Module` 类还有一个实例方法和类方法，方法名都叫 `constants`，`Module#constants` 方法返回当前范围内的所有常量，`Module.constants` 方法返回当前程序中所有顶层的常量。

```ruby
M.constans # => [:C, :Y]
Module.constants.include? :Object # => true
Module.constants.include? :Module # => true
```

`Module.nesting` 告诉我们当前代码所在的路径：

```ruby
module M
  class C
    module M2
      Module.nesting # => [M::C::M2, M::C, M]
    end
  end
end
```

### 对象和类小结

- 对象是一组实例变量 + 一个指向类的引用
- 对象的方法存放在对象的类中，对于类来说，这些方法被称为类的实例方法
- 类是一个对象（Class 类的一个实例） + 一组实例方法 + 一个对其超类的引用
- 类有自己的方法（也就是 Class 的实例方法，比如 new）
- Class 是 Module 的子类

### 使用命名空间解决冲突的问题

比如你创建一个 Text 类，但 Action Mailer 中已经有了一个 Text Module，你可以使用命名空间来解决冲突：

```ruby
module MyModule
  class Text
  end
end
```

 

### 调用方法时发生了什么

#### 方法查找

- 接受者 receiver：调用方法所在所在的对象
- 祖先链 ancestors chain：找到对象的类，再找到类的超类，再找超类的超类，直到找到 `BasicObject` 类

```ruby
class MyClass
  def my_method
    'my_method()'
  end
end

class MySubclass < MyClass
end

obj = MySubclass.new
obj.my_method() # => 'my_method()'

MySubclass.ancestors # => [MySubclass, MyClass, Object, Kernel, BasicObject]
```

`include` 某个模块或者继承某个类的时候，会将其加入到祖先链中。`prepend` 与 `include` 相似，前者会将模块插入到（祖先链中）该类的下方，后者会插入上方

##### Kernel 模块

我们可以随时调用比如 `print` 等方法，但其实 `print` 方法是 Kernel 模块的私有实例方法，Object 类包含了 Kernel 模块，使得每个对象都可以调用 Kernel 模块的方法

#### 执行方法

##### self

`self` 代表了当前对象，当调用一个方法时，接受者就是 `self`

```ruby
class MyClass
  def testing_self
    @var = 10   # self 的一个实例变量
    my_method() # 相当于 self.my_method()
    self.my_private_method() # 不能这样写，private 方法不能指定接受者，只能通过隐式的 self 调用
    self
  end
  
  def my_method
    @var = @var + 1
  end
  
  private
  
  def my_private_method
  end
end

obj = MyClass.new
obj.testing_self # => #<MyClass:0x123 @var=11>
```

````ruby
self       # => main
self.class # => Object

class MyClass
  self     # => MyClass，在所有方法之外，self 是这个类或模块本身
end
````

#### 细化 Refinement

```ruby
module StringExtensions
  # 为 String 类 refine 了一个 to_alphanumeric 方法，他默认不生效，不能直接通过 String#to_alphanumeric 来调用
  refine String do
    def to_alphanumeric
      gsub(/[^\w\s]/, '')
    end
  end
end

module StringStuff
  using StringExtensions
  "my_string".to_alphanumeric  # => 可以调用
end

"my_string".to_alphanumeric    # => 不能调用
```

细化只有在两种场合有效：

1. `refine` 代码内部
2. `using` 语句开始到模块结束（或者在顶层作用域调用到文件结束）

