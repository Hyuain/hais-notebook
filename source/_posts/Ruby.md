---
title: Ruby
date: 2022-03-13 10:05:22
tags:
  - 入门
  - Ruby
categories:
  - [全栈, Ruby]
---

结合网课、Ruby 文档和《Ruby 元编程 2》。

「元编程是编写能在运行时操作语言构件的代码。」——能写代码的代码Ruby 的介绍与基础。

<!-- more -->

# Startup

## 运行 Ruby

- `irb` 可交互式命令行
- `ruby` + 文件路径

# Basic Syntax

## 值

### 变量

- 没有 `var` `const` `let` 等关键字
- 局部变量：小写字母开头或 `_` 开头
- 全局变量：`$` 开头
- 类变量：`@@` 开头
- 实例变量：`@` 开头

### 常量

- 任何以大写字母开头的引用（包括类名和模块名）都是常量
- 我们可以修改常量的值，尽管会得到一个警告

- 常量和变量的最大区别在于作用域不同，常量在所有作用域中都能访问，而不同作用域中的变量则是分开的（变量与作用域见 Blocks 章节）

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

#### 使用命名空间解决冲突的问题

比如你创建一个 Text 类，但 Action Mailer 中已经有了一个 Text Module，你可以使用命名空间来解决冲突：

```ruby
module MyModule
  class Text
  end
end
```

### 多重赋值

- `a,b,c = 1,2,3`
- `a,b = [1,2]`
- `a,b,*c = 1,2,3,4,5`
- `a,*b,c = 1,2,3,4,5`
- `a,b = b,a`

### 字符串

- 单引号，没有转义，写 `'\n'` 里面就是 `\n`
- 双引号，有转义
- 多行字符串

```ruby
s = <<eos
dfa123
fad
eos
```

## 数据类型

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

### Array

`arr.methods` 可以看到有什么 api

### Hash

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

## 语句

### 控制语句

#### if

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

#### case

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

### 循环语句

#### times

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

#### each

```ruby
[1, 2, 3].each do |item| p item end
```

```ruby
(1..7).each do |n| p n end
```

#### for

```ruby
for i in 1..7 do p i end
```

#### while/until

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

#### loop

一直循环

```ruby
loop
  p 'ruby'
  if x > 10 then break
end
```

#### break continue

`break` 和 `next` 对应 `break` 和 `continue`

## 其他

### log

- `print`
- `puts` put string
- `p`

### 注释

- 单行注释 `#`
- 多行注释

```ruby
=begin
注释
=end
```

# Metaprogramming

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

# The Object Model

Ruby 中，有很多语言构件：对象、类、模块、实例变量等，元编程操作的就是这些语言构件。所有这些语言构件存在的系统称之为**对象模型**。

- **对象** 是一组 **实例变量** + 一个 **指向类的引用**（`.class`）
- 对象的方法存放在对象的类中，对于类来说，这些方法被称为类的实例方法
- **类 **是一个 **对象**（Class 类的一个实例） + 一组 **实例方法** + 一个 **对其超类的引用**（`.superclass`）
- 类有自己的方法（也就是 Class 的实例方法，比如 new）
- Class 是 Module 的子类

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

### 实例变量与实例方法

```ruby
class MyClass
  def my_method
    @v = 1
  end
end

obj = MyClass.new
```

1. 我们用 `#` 指类的实例方法，`.` 代指类的方法
2. 实例变量 `[:@v]`：通过 `Object#instance_variables` 方法来查看。对象的类与实例变量没有关系，当给实例变量赋值时，他们就突然出现了。
3. 实例方法 `[:my_method]`：通过 `Object#methods` 方法查看，他会给出所有继承的方法，我们可以通过 `Array#grep` 方法进行过滤。

> 实例变量存在对象中，而方法存放在类中。

{% note warning %}
需要注意的是，我们不能说 `my_method` 是 `MyClass` 的方法，因为我们不能通过 `MyClass.my_method()` 这样来调用它。

我们只能说 `my_method` 是 `MyClass` 的一个 **实例方法**，因为我们必须要先定义一个 `MyClass` 的实例，然后通过实例调用它。

```ruby
String.instance_methods == "abc".methods  # => true
String.methods == "abc".methods  # => false
```

{% endnote %}

### 类本身也是对象

类本身也是对象，相当于是 `Class` 的实例，这又有点像 JavaScript 了

```ruby
"hello".class # => String
String.class  # => Class
```

根据前面所述，**实例的方法就是类的实例方法**。因此，一个类的方法就是 `Class` 的实例方法

```ruby
# false 表示忽略继承的方法
Class.instance_methods(false) # => [:allocate,:new,:superclass]
```

`allocate` 方法是 `new` 方法的支撑方法，`superclass` 则跟继承有关：

```ruby
Array.superclass  	 	# => Object
Object.superclass 		# => BasicObject
BasicObject.superclass  # => nil
Class.superclass  		# => Module
Module.superclass       # => Object
```

`Class` 的 `superclass` 是 `Module`，类其实就是带有 `new` `allocate` `superclass` 的增强模块，这三个方法可以让我们按照一定层次创建对象。

- 如果希望把代码 `include` 到别的代码中，就应该使用模块
- 如果希望代码被实例化或继承，应该使用类

### 方法调用

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

# Methods

## 动态方法 Dynamic Methods

### 动态调用方法

> 实际上调用一个方法就是给一个对象发送消息

除了使用点标识符 `(.)` 来调用方法，还可以使用 `Object#send` 方法来调用某个实例的方法：

```ruby
class MyClass
  def my_method(my_arg)
    my_arg * 2
  end
end

obj = MyClass.new
obj.my_method(3)        # => 6
obj.send(:my_method, 3) # => 6
# send 的第一个参数是方法名，可以使用字符串，也可以使用符号；剩下的参数和代码块会被直接传递给调用的方法
```

这样动态调用方法的技巧叫做 **动态派发（Dynamic Dispatch）**

### 动态定义方法

可以用 `Module#define_method()` 动态定义方法：

```ruby
class MyClass
  # 与 def 不同，define_method 允许在运行时决定方法的名字
  define_method :my_method do |my_arg|
    my_arg * 3
  end
end

obj = MyClass.new
obj.my_method(2)    # => 6
```

## method_missing

当找不到方法是时，他就会调用对象上的 `method_missing` 方法，而我们可以覆写这个方法。调用者看起来调用了某个方法，但实际上这个方法却不存在，这称为 **幽灵方法（Ghost Method）**

### respond_to_missing

如果我们通过幽灵方法实现某功能，我们会发现使用 `respond_to?` 来查找方法的时候就会发现找不到这个方法。

需要使用 `respond_to_missing` 方法才能感知到幽灵方法。因此我们通常在使用 `method_missing` 的时候会覆写 `respond_to_missing` 方法

### const_missing

当引用一个不存在的常量的时候，Ruby 会把这个常量名作为一个符号传给 `const_missing` 方法，我们可以通过覆盖 `const_missing` 方法达到拦截的目的：

```ruby
require 'rake'
task_class = Task  # => 得到一个警告，让你使用 Rake::Task
```

能达到上面的效果是因为 Rake 覆盖了 `Module#const_missing`：

```ruby
class Module
  def const_missing(const_name)
    case const_name
    when :Task
      Rake.application.const_warning(const_name)
      Rake::Task
    end
  end
end
```

## 白板类 Blank Slates

用幽灵方法时经常会遇到名称与已经存在的方法冲突的情况，尤其是与他继承来的方法冲突。这时候就需要删掉继承的很多无用的方法。

### BasicObject

`BasicObject` 只有几个实例方法，我们直接创建的类默认继承自 `Object` 类（`BasicObject` 的子类），我们可以将其改为继承 `BasicObject` 类

### 删除方法

1. `Module#undef_method`：删除（包括继承来的）方法
2. `Module#remove_method`：删除接受者自己的方法

#  Blocks

代码块源自像 LISP 这样的函数式编程语言

## 代码块的基础使用

```ruby
def a_method(a, b)
  a + yield(a, b)
end

a_method(1, 2) { |x, y| ( x + y ) * 3 }   # => 10
```

- 代码块可以用 `do...end` 定义，也可以用大括号定义
- 只有在调用一个方法的时候才能定义一个块
- 这个块会被直接传递给这个方法，该方法可以用 `yield` 关键字调用这个块
- 在一个方法里，可以通过 `Kernel#block_given?` 询问当前方法的调用是否包含块

## 闭包与作用域

### 代码块是闭包

可以运行的代码 = 代码本身 + 绑定

```ruby
def my_method
  x = "Good bye"
  yield("cruel")
end

x = "Hello"
my_method { |y| "#{x}, #{y} word" }  # => Hello, cruel world
# 创建代码块时，x 会成为局部绑定，然后把代码块连同绑定传给 my_method 方法
# 代码块中看到的 x 不是方法里面的 x，而是创建代码块时绑定的 x
```

```ruby
def just_yield
  yield
end

top_level_variable = 1

just_yield do
  top_level_variable += 1
  local_to_block = 1  # => 在代码块中定义的额外的绑定
end

top_level_variable  # => 2
local_to_block      # => Error，看不到了，额外的绑定在代码块结束时消失了
```

因此代码块，又被称为 **闭包（Closure）**

### 作用域

Ruby 中没有内部作用域和外部作用域的概念（即在内部作用域中可以看到外部作用域的变量）。Ruby 中不同的作用域是分开的，没有“内部作用域可以看到外部作用域的变量”这样的说法。

```ruby
v1 = 1
class MyClass
  v2 = 2
  local_variables # => [:v2]
  def my_method
    v3 = 3
    local_variables
  end
  local_variables # => [:v2]
end

obj = MyClass.new
obj.my_method # => [:v3]
local_variables # => [:v1, :obj]
```

#### 全局变量和顶级实例变量

全局变量

- 全局变量可以在任何作用域中访问
- 变量名最开始有 `$`
- 所有人都可以修改全局变量

```ruby
def a_scope
  $var = "some value"
end

def another_scope
  $var
end

a_scope
another_scope  # => "some value"
```

顶级实例变量

- 顶级对象 `main` 的实例变量
- 有时可以用来代替全局变量
- 一般认为顶级实例变量更安全，但也不是完全安全

```ruby
@var = "the top-level @var"

def my_method
  @var
end

my_method	 # => "the top-level @var"

def MyClass
  def my_method
    @var = "this is not the top-level @var"
    # 由于这里的 self 不是 main，因此他不是顶级实例变量
  end
end
```

#### 作用域门

在这三种情况下，程序会关闭前一个作用域，打开一个新的作用域：

- 类定义 `class`
- 模块定义 `module`
- 方法 `def`

`class` `module` `def` 是切换作用域的标志，每个关键字都对应一个 **作用域门（Scope Gate）**。

另外注意类和模组定义中的代码会立即自动执行一次，而方法定义中的则不会。

#### 扁平化作用域

通过替换掉 `class` `def` 这些关键字，来让变量穿越作用域门，这种技巧叫嵌套文法作用域（nested lexical scopes），也称扁平化作用域（flattening the scope）。

```ruby
my_var = "Success"

# 用 Class.new 代替 class
MyClass = Class.new do
  puts "#{my_var} in the class definition"
  
  # 用 define_method 代替 def
  define_method :my_method do
    puts "#{my_var} in the method"
  end
end

MyClass.new.my_method
# => Success in the class definition
# => Success in the method
```

#### 共享作用域

利用扁平化作用域，可以实现在一组方法之间共享某个变量，而其他方法则无法访问这个变量。

```ruby
def define_methods
  shared = 0
  # 定义 Kernel#counter 方法
  Kernel.send :define_method, :counter do
    shared
  end
  Kernel.send :define_method, :inc do
    shared += x
  end
end

define_methods

counter 	# => 0
inc(4)
counter   # => 4
```

## `instance_eval`

```ruby
class MyClass
  def initialize
    @v = 1
  end
end

obj = MyClass.new

obj.instance_eval do
  self	# => #<MyClass:0x00000000062acbb8 @v=1>
  @v		# => 1
end
```

传给 `instance_eval` 中代码块中的 `self` 就是 `obj`，并且在这个代码块中可以使用并修改实例变量 `@v`。

因此把传给 `BasicObject#instance_eval` 方法的代码块称为  **上下文探针（Context Prob）**，他就像一个深入到对象中的代码片段，并且可以对这个对象进行操作。

`instance_eval` 会破坏封装，可以肆意查看私有数据，但有时在开发和测试中使用会非常方便。

### `instance_exec`

```ruby
class C
  def initialize
    @x = 1
  end
end

class D
  def twisted_method
    @y = 2
    # 注意这样打印的 @y 是 nil，因为代码块已经不在 twisted_method 的作用域里面了
    C.new.instance_eval { p "@x: #{@x}, @y: #{@y}" }
    # 可以使用 instance_exec 方法将 @y 传进去
    C.new.instance_exec(@y) { |y| p "@x: #{@x}, @y: #{y}" }
  end
end

D.new.twisted_method
```

## 可调用对象

代码块的使用分为：1. 打包代码；2. 通过 `yield` 语句执行代码。

除了代码块之外，还有三种打包代码的方式：

1. proc，proc 是由块转换来的对象
2. lambda，proc 的变种
3. 使用方法

### Proc

#### 创建 Proc

##### Proc.new

将代码块传给 `Proc.new` 就可以创建一个 `Proc`

```ruby
inc = Proc.new { |x| x + 1 }
inc.call(2)		# => 3
```

##### lambda

用 `Kernel#lambda` 方法也可以创建 `Proc`

```ruby
dec = lambda { |x| x - 1 }
# 上述代码也可以这样写
dec = ->(x) { x - 1 }
dec.class 	  # => Proc
dec.call(2)	  # => 1
```

##### & 操作符

通过给方法添加一个特殊的参数，可以将代码块附加到一个绑定上，这个参数必须满足：

1. 是参数列表的最后一个参数
2. 以 & 开头

```ruby
def math(a, b)
  yield(a, b)
end

def do_math(a, b, &operation)
  # 将 & 去掉，就可以得到一个 Proc 对象；也可以将一个 Proc 加上 & 得到一个代码块
  p operation.class    # => Proc
  math(a, b, &operation)
end

do_math(2, 3) { |x, y| x * y }		# => 6
```

#### 执行 Proc

然后通过 `Proc#call` 方法执行 `Proc` 对应的代码块。

这种技巧被称为 **延迟执行（Deferred Evaluation）**：还可以使用 `lambda` 方法创建 Proc：#### & 操作符

### Proc 和 Lambda

用 `lambda` 方法创建的 `Proc` 被称为 `lambda`， 其他方式创建的则称为 `proc`，可以通过 `Proc#lamda?` 方法来检测 `Proc` 是不是 `lambda`。

#### Proc、Lambda 和 return

`lambda` 和 `proc` 的 `return` 关键字含义不同。

- `lambda` 中的 `return` 仅仅表示从这个 `lambda` 中返回
- `proc` 中的 `return` 表示从定义 `proc` 的作用域中返回

```ruby
def double(callable_object)
  callable_object.call * 2
end

l = lambda { return 10 }
double(l)			# => 20
```

```ruby
def another_double
  p = Proc.new { return 10 }
  result = p.call
  return result * 2		# 不可到达的代码
end

another_double # => 10
```

因此下面代码是错误的：

```ruby
def double(callable_object)
  callable_object.call * 2
end

p = Proc.new { return 10 }
double(p)			# => 不可到达的代码
```

可以不使用 `return` 来规避这个问题

#### Proc、Lambda 和参数数量

如果给一个定义了两个参数的 `proc` 或 `lambda` 传一个或三个参数，可能会导致错误，并且 `lambda` 的适应能力比 `proc` 和普通代码块差。

如果参数比期望的多，`proc` 会忽略多余的参数；如果参数数量不足，对于未指定的参数，`proc` 会赋值为 `nil`。

整体而言，`lambda` 更直观，因为他更像一个方法；因此许多 Ruby 程序员优先使用 `lambda`。

### Method

1. 通过 `Kernel#method` 方法可以获得一个用 Method 对象表示的方法，可以用 `Method#call` 来调用这个方法。

2. 可以通过 `Kernel#singleton_method` 方法把 **单件方法** 名转换为 Method 对象。
3. 可以通过 `Method#to_proc` 方法把 Method 转换为 Proc
4. 可以通过 `define_method` 方法把代码块转换为方法
5. 注意 `lambda` 在定义他的作用域中执行（闭包），Method 对象则在他自身所在对象的作用域中执行

```ruby
class MyClass
  def initialize(value)
    @x = value
  end
  def my_method
    p @x
  end
end

obj = MyClass.new(1)
m = obj.method :my_method
m.call 		# => 1
```

#### 自由方法

自由方法（unbound method）跟普通方法类似，但他从定义它的类或模块中脱离了。

通过 `Method#unbind` 方法可以讲一个方法变成自由方法，也可以直接调用 `Module#instance_method` 方法获得自有方法：

```ruby
module MyModule
  def my_method
    42
  end
end

unbound = MyModule.instance_method(:my_method)
unbound.class			# => UnboundMethod
```

不能调用 UnboundMethod，但可以将它绑到一个对象上，让他再次成为 Method 对象。

可以通过 `UnboundMethod#bind` 方法，也可以通过 `Module#define_method` 方法来进行绑定。

```ruby
String.send :define_method, :another_method, unbound
"abc".another_method		# => 42
```

# Class

## 类的定义

### 当前类

不管在哪里，都会有一个当前对象：`self`，和一个当前类（或模块）。定义一个方法时，那个方法将成为当前类的一个实例方法。

- 在程序的顶层，当前类是 `Object`，这是 `main` 对象所属的类。在程序顶层定义的方法会成为 `Object` 实例方法。
- 在一个方法中，当前类就是当前对象的类。

```ruby
class C
  def m1
    def m2; end
  end
end

class D < C; end

obj = D.new
obj.m1

C.instance_methods(false)    	# => [:m1, :m2]
```

- 当用 `class` 关键字打开一个类时（或用 `module` 关键字打开模块时），这个类成为当前类。

#### `class_eval`

通过 `Module#class_eval`方法（别名 `module_eval` 方法），可以不使用 `class` 关键字就修改某个类，在不知道某个类的名字的时候比较好用：

```ruby
def add_method_to(a_class)
  a_class.class_eval do
    def m; 'Hello!'; end
  end
end

add_method_to String
"abc".m				# => "Hello!"
```

- 与 `Object#instance_eval` 方法不同，`instance_val` 只修改 `self`，但 `class_eval` 会修改当前类和 `self`。
- `class_eval` 与 `class` 不同，并不会打开一个新的作用域，而可以使用外部的变量。
- 另外还有 `module_exec` 和  `class_exec`，可以接受额外的参数。

### 类实例变量



