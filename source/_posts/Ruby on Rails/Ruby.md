---
title: Ruby
date: 2022-03-13 10:05:22
categories:
  - - 全栈
tags:
  - RB1
---

结合网课、Ruby 文档和《Ruby 元编程 2》。

「元编程是编写能在运行时操作语言构件的代码。」

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

`arr.methods` 可以看到有什么 api。

```ruby
array = [10, 20]
element = 30
array << element # => [10, 20, 30]
```

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
- 祖先链 ancestors chain：找到对象的类（class）*，再找到类的超类（superclass），再找超类的超类（superclass），直到找到 `BasicObject` 类

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

`include` 某个模块或者继承某个类的时候，会将其加入到祖先链中。`prepend` 与 `include` 相似，前者会将模块插入到（祖先链中）该类的下方，后者会插入上方。

{% note warning %}

*事实上，查找方法的时候会先找该对象的 **单件类（singleton_class）**，再找 **单件类的超类**（也就是该对象的类）。但单件类并不会被 `class` 、`ancestors` 方法显示出来，详情请看单件类章节。

{% endnote %}

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
  self											# => #<MyClass:0x00000000062acbb8 @v=1>
  @v												# => 1
  def my_method!; end       # => 这样定义的方法是 obj 的单件方法
end
```

- 传给 `instance_eval` 中代码块中的 `self` 就是 `obj`
- 并且在这个代码块中可以使用并修改实例变量 `@v`
- 这个代码块中的当前类是接受者的 **单件类**
- `instance_eval` 表示：我想修改 `self`

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

another_double 				# => 10
```

因此下面代码是错误的：

```ruby
def double(callable_object)
  callable_object.call * 2
end

p = Proc.new { return 10 }
double(p)			# 不可到达的代码
```

可以不使用 `return` 来规避这  个问题

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

- 与 `Object#instance_eval` 方法不同：
  - `instance_val` 修改 `self` ，并且会把当前类修改为接受者的单件类；
  - `class_eval` 会修改当前类和 `self`。

- `class_eval` 与 `class` 不同，并不会打开一个新的作用域，而可以使用外部的变量。
- 另外还有 `module_exec` 和  `class_exec`，可以接受额外的参数。

### 类实例变量

类定义中的 `self` 是类本身，因此不要混淆了 **类的实例变量** 和 **类的对象的实例变量**

```ruby
 class MyClass
   @my_var = 1
   def self.read; @my_var; end
   def write; @my_var = 2; end
   def read; @my_var; end
 end

obj = MyClass.new
obj.read			# => nil
obj.write
obj.read			# => 2
MyClass.read  # => 1
```

### 类变量

类变量可以被子类或类的实例所使用。

```ruby
class C
  @@v = 1
end

class D < C
  def my_method; @@v; end
end

D.new.my_method			# => 1
```

因为类变量不属于这个类，而是属于类体系结构，因此会产生一些别的问题：

```ruby
@@v = 1

class MyClass
  @@v = 2
end

# @@v 属于 main 的类 Object，所以也属于 Object 的所有后代，MyClass 继承自 Object，也共享了这个变量
@@v 	 # => 2
```

## 单件方法

只对单个对象生效的方法，称为 **单件方法（Singleton Method）**。

```ruby
str = "just a regular string"

def str.title?
  self.upcase == self
end

str.title?										# => false
str.methods.grep(/title?/)		# => [:title?]
str.singleton_methods					# => [:title?]
```

### 类方法

类方法其实就是一个类的单件方法。

```ruby
def obj.a_singleton_method; end
def MyClass.another_class_method; end

# 定义单件方法
def object.method
  # 方法的主体
end
```

一共有三种方式可以定义类方法：

```ruby
def MyClass.a_class_method; end

class MyClass
  def self.another_class_method; end
end

class MyClass
  class << self
    def yet_another_class_method; end
  end
end
```

### 类宏

#### `attr_accessor`

Ruby 的对象是没有属性的，如果想要得到 JavaScript 类似的属性的效果，得定义 **拟态方法**：

```ruby
class MyClass
  def my_attribute=(value)
    @my_attribute = value
  end
  def my_attribute
    @my_attribute
  end
end

obj = MyClass.new
obj.my_attribute = 'x'
obj.my_attribute          # => "x"
```

这样的写法（也叫访问器）很麻烦，可以用 `Module#attr_*` 方法定义访问器，`Module#attr_reader` 生成读方法、`Module#attr_writer` 生成写方法、`Module#attr_accessor` 同时生成两者：

```ruby
class MyClass
  attr_accessor :my_attribute
end
```

所有的 `attr_*` 方法都定义在 `Module` 类中，因此不管 `self` 是类还是模块，都可以使用。这种方法被称为 **类宏（Class Macro）**，简单来说就是可以在类定义中使用的方法。

下面是一个使用类宏来标记方法名已弃用的例子：

```ruby
class Book
  def title #...
  def subtitle #...
  def lend_to #...
  
  def self.deprecate(old_method, new_method)
    define_method(old_method) do |*args, &block|
      warn "Warning #{old_method}() is deprecated. Use #{new_method}()"
      send(new_method, *args, &block)
    end
  end
  
  deprecate :GetTitle, :title
  deprecate :LEND_TO_USER, :lend_to
  deprecate :title2, :subtitle
end
```

## 单件类

根据之前所学，查找方法时会先进入接受者的类，然后沿着 superclass 向上查找。但是这里面都没有容纳对象单件方法的地方。

事实上，每个对象都有一个特有的隐藏类（这个类不能用 `Object#class` 方法查看到），被称为该对象的单件类，也被称为元类（metaclass）或本征类（eigenclass）。

可以通过一个 **特殊的语法** `class << my_object` 进入该单件类的作用域，并获得他的引用：

```ruby
class C
  def a_method
    'C#a_method()'
  end
end

class D < C; end

obj = D.new

singleton_class = class << obj
  # 这里是你的代码
  # 这里定义的方法 = 单件类的实例方法 = 单件方法
  def a_singleton_method
    'obj#a_singleton_method'
  end
  
  # 可以返回 self，让外面得到单件类的引用
  self
end

singleton_class.class				 # => Class
singleton_class.superclass 	 # => D，对象的单件类的超类 = 对象的类
```

还可以通过 `Object#singleton_class` 方法来获得单件类的引用：

```ruby
"abc".singleton_class		# => #<Class:#<String:0x0000000006df44a8>>
```

**每个单件类只有一个实例，单件类不能被继承，单件方法就在单件类中：**

```ruby
def obj.my_singleton_method; end
singleton_class.instance_method.grep(/my_/)    # => [:my_singleton_method]
```

### 单件类和继承

```ruby
class C
  class << self
    def a_class_method
      'C.a_class_method()'
    end
  end
end

class D < C; end

C.singleton_class							# => #<Class:C>，注意单件类前面有#
D.singleton_class   					# => #<Class:D>
D.singleton_class.superclass 	# => #<Class:C>，注意类的单件类的 superclass 与普通对象的单件类的 superclass 不同
C.signleton_class.superclass	# => #<Class:Object>
```

![Ruby 单件类和继承](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Ruby%20%E5%8D%95%E4%BB%B6%E7%B1%BB%E5%92%8C%E7%BB%A7%E6%89%BF.jpg)

{% note warning %}

一个对象的单件类的超类是这个对象的类。

一个类的单件类的超类是这个类的超类的单件类。

{% endnote %}

`Kernel` 模块置于 `Object` `BasicObnject` 之间，他也有自己的单件类，但他的单件类并不是 `obj` 或者 `#D` 祖先链上的一员。

用这种组织方式，可以在子类上调用父类的类方法：

```ruby
D.a_class_method 		# => "C.a_class_method()"
```

> 其实单件类也有自己的单件类，暂时不用管这个。

### 单件类的用处

给一个类定义类属性：

```ruby
# 这样定义的话，b 实际上在 Class 上，所有的类都会增加 b
class MyClass; end
class Class
  attr_accessor :b
end
MyClass.b = 42
MyClass.b				# => 42

# 可以定义到 MyClass 的单件类上
class MyClass
  class << self
    attr_accessor :c
  end
end
MyClass.c = "It works!"
MyClass.c				# => "It works!"
```

### 类扩展

类扩展：通过 `include` 的方式为类添加单件方法。

尝试在模块中定义一个单件方法，然后通过 `include` 的方式让类可以访问到他，失败了：

```ruby
class C
  def self.my_method; 'hi'; end
end

module M
  def self.my_method; 'hello'; end
end

class D < C; end

class E
  include M
end

p D.my_method    # => ‘hi'
p E.my_method	   # => NoMethodError!
```

因为 `include` 获得的是该模块的实例方法，这与继承有点区别。

如果要达到想要的效果，需要在模块中定义普通的实例方法，然后让类的单件类来包含这个模块：

```ruby
module M
  def my_method; 'hello'; end
end

class E
  class << self
    include M
  end
end

MyClass.my_method			# => 'hello'
```

### 对象扩展

对象扩展：用类扩展的技巧，为对象添加单件方法。

```ruby
module M
  def my_method; 'hello'; end
end

obj = Object.new

class << obj
  include M
end

obj.my_method							# => 'hello'
obj.singleton_methods			# => [:my_method]
```

可以用 `Object#extend` 方法来更好地书写类扩展和对象扩展：

```ruby
module M
  def my_method; 'hello'; end
end

obj = Object.new
obj.extend M
obj.my_method					# => 'hello'

class C
  extend M
end

C.my_method						# => 'hello'
```

## 方法包装器

有三种方式可以用一个方法包装另一个方法。

### 方法别名与环绕别名

可以用 `alias` 关键字或 `Module#alias_method` 来给方法取一个新名字

```ruby
class MyClass
  def my_method; 'my_method()'; end
  alias_method :m, :my_method
end

obj = MyClass.new
obj.my_method			# => "my_method()"
obj,m							# => "my_method()"
```

重新定义方法的时候，并不会真正修改原来的方法，因此如果给一个方法取了别名之后，重新定义一个跟之前名字相同的方法，别名仍然绑定的原来的方法。

```ruby
class String
  alias_method :real_length, :length
  
  def length
    real_length > 5 'long' : 'short'
  end
end

"War and Peace".length 		   # => "long"
"war and Peace".real_length  # => 13
```

可以用 **环绕别名** 的技巧来包装一个一个方法：

1. 给方法定义一个别名
2. 重新定义这个方法
3. 在新方法中调用老方法

{% note warning %}

环绕别名实际上也是一种猴子补丁，他会全局性地破坏已有代码。

{% endnote %}

### 细化封装器

在细化的方法中调用 `super` 方法，就会调用没有细化的原始方法：

```ruby
module StringRefinement
  refine String do
    def length
      super > 5 ? 'long' : 'short'
    end
  end
end

using StringRefinement

"War and Peace".length 	 # => "long"
```

同其他细化一样，细化封装器的作用范围只到文件末尾处（Ruby 2.1 中是模块定义的范围内），因此比环绕别名更安全。

### `Module#prepend` 与下包含包装器

与 `include` 类似，但是 `Module#prepend` 方法会将包含的模块插在祖先链中该类的下方，而非上方。因此可以用这个技巧复写同名方法，也可以用 `super` 调用到该类的原始方法。

```ruby
module ExplicitString
  def length
    super > 5 ? 'long' : 'short'
  end
end

String.class_eval do
  prepend ExplicitString
end

# 也可以用打开类的方式书写
class String
  prepend ExplicitString
end

"War and Peace".length 	 # => "long"
```

这种技巧被称为下包含包装器。

# Code That Writes Code

## `Kernel#eval`

使用 `Kenerl#eval` 方法可以直接执行包含 Ruby 代码的字符串：

```ruby
array = [10, 20]
element = 30
eval("array << element")    # => [10, 20, 30]
```

比如在 REST Client 中，通过这种方式批量定义这些方法：

```ruby
# 我们需要定义形如这样的方法
def get(path, *args, &b)
  r[path].get(*args, &b)
end

# 可以这样使用 eval 定义
POSSIBLE_VERBS = ['get', 'put', 'post', 'delete']

POSSIBLE_VERBS.each do |m|
  # 这种方式被称为 here 文档（here document, heredoc）
  # 在 eval 后面接一个字符串，<< 后面紧跟一个结束序列（terminal sequence）（在这里是 end_eval）
  # 下次再遇到结束列的时候，字符串定义结束
  eval <<-end_eval
		def #{m}(path, *args, &b)
			r[path].#{m}(*args, &b)
		end
	end_eval
end
```

### 绑定对象

`Binding` 就是一个用对象表示的完整作用域，可以通过 `Binding` 对象来捕获并带走当前的作用域，然后通过 `eval` 方法在这个 `Binding` 对象所携带的作用域中执行代码。

可以用 `Kernel#binding` 方法来创建 `Binding` 对象：

```ruby
class MyClass
  def my_method
    @x = 1
    binding
  end
end

b = MyClass.new.my_method
```

`Binding` 对象可以看做是比代码块更“纯净”的闭包，因为他只包含作用域，不包含代码。

可以将 `Binding` 对象传给 `eval` 方法：

```ruby
eval "@x", b  	# => 1
```

Ruby 中还有一个常量 `TOPLEVEL_BINDING`，表示顶级作用于的 `Binding` 对象：

```ruby
class AnotherClass
  def my_method
    eval "self", TOPLEVEL_BINDING
  end
end

AnotherClass.new.my_method		# => main
```

### 应该选择哪种 `*eval`

现在有三种 `*eval`：`eval`、`instance_eval` 和 `class_eval`：

1. `eval` 只能执行代码字符串，不能执行代码块；`instance_eval` 和 `class_eval` 除了执行代码块，也可以执行代码字符串

2. 代码字符串中也可以像块那样访问局部变量：

   ```ruby
   array = ['a', 'b', 'c']
   x = 'd'
   array.instance_eval "self[1] = x"
   
   array   # => ["a", "d", "c"]
   ```

3. 能使用代码块就尽量用代码块，代码字符串安全性会有问题

#### 代码注入

比如你写了一个方法，检查数组是否存在某方法：

```ruby
def explore_array(method)
  code = "['a', 'b', 'c'].#{method}"
  puts "Evaluating: #{code}"
  eval code
end
```

结果用户输入了这样一个字符串：

```ruby
object_id; Dir.glob(*)
# 就会执行
['a', 'b', 'c'].object_id; Dir.glob("*")		# => 你的私有信息就会暴露在这里
```

#### 防止代码注入

1. 完全禁止 `eval`，根据具体的问题寻找替代方法，比如动态方法、动态派发
2. 用下面的一些方法更安全地使用 `eval`

#### 污染对象和安全级别

Ruby 会自动将不安全的对象（尤其是外部传入的对象）标记为污染对象，可以用 `Object#tainted?` 方法来判断一个对象是不是被污染了：

```ruby
user_input = "User input #{gets()}"
puts user_input.tainted?			# => true
```

可以通过给 `$SAFE` 全局变量赋值来设置一个安全级别，就可以禁止某些危险操作：

- 默认 0，什么都可以做；在任何大于 0 的安全级别上，Ruby 都会拒绝执行污染的字符串
- 2 可以禁止绝大多数与文件相关的操作
- 最高 3，创建的每一个对象都是被污染的

可以通过 `Object#untain` 方法来手动去除字符串的污染性。

**沙盒（Sandbox）**指的就是通过使用安全级别为 `eval` 方法创造的一个可控环境。

比如 ERB 中对 HTML 模板中写的 Ruby 代码，会创建一个沙盒来执行：

```ruby
class ERB
  # new_toplevel 是顶级变量 TOPLEVEL_BINDING 的一个拷贝
  def result(b=new_toplevel)
    if @safe_level
      # 使用 Proc 作为一个洁净室
      proc {
        # 这个安全级别只有 proc 中有效
        $SAFE = @safe_level
        eval(@src, b, (@filename || '(erb)'), 0)
      }.call
    else
      eval(@src, b, (@filename || 'erb'), 0)
    end
  end
  #...
```

## 钩子方法

有不少钩子方法，可以通过覆写这些方法在特定的时间完成一些操作，比如：

```ruby
class String
  def self.inherited(subclass)
    puts "#{self} was inherited by #{subclass}"
  end
end

module M1
  def self.included(othermod)
    puts "M1 was included into #{othermod}"
  end
end

module M2
  def self.prepended(othermod)
    puts "M2 was prepended to #{othermod}"
  end
end
```

另外还有 `Module#extend_object` `method_added` `method_removed` `undefined` 等方法。

如果需要捕获单件方法，则需要使用 `singleton_method_added` `singleton_methoded_removed` 等方法。

## 例子

需要编写一个 `add_checked_attribute` 方法，为类添加一个经过校验的属性，下面是测试用例：

```ruby
require 'test/unit'

class Person; end

class TestCheckedAttribute < Test::Unit::TestCase
  def setup
    add_checked_attribute(Person, :age)
    @bob = Person.new
  end
  
  def test_accepts_valid_values
    @bob.age = 20
    assert_equal 20, @bob.age
  end
  
  def test_refuses_nil_values
    assert_raises RuntimeError, 'Invalid attribute' do
      @bob.age = nil
    end
  end
  
  def test_refuses_false_values
    assert_raises RuntimeError, 'Invalid attribute' do
      @bob.age = false
    end
  end
end
```

### 使用 `eval` 来完成

```ruby
def add_checked_attribute(klass, attribute, &validation)
  eval "
		class #{klass}
			def #{attribute}=(value)
				raise 'Invalid attribute' unless value
				@#{attribute} = value
			end

			def #{attribute}()
				@#{attribute}
			end
		end
  "
end
```

### 去掉 `eval`

```ruby
def add_checked_attribute(klass, attribute)
  klass.class_eval do
    define_method "#{attribute}=" do |value|
      raise 'Invalid attribute' unless value
      instance_variable_set("@#{attribute}", value)
    end
    
    define_method attribute do
      instance_variable_get "@#{attribute}"
    end
  end
end
```

### 通过一个代码块来实现校验

```ruby
require 'test/unit'

class Person; end

class TestCheckedAttribute < Test::Unit::TestCase
  def setup
    add_checked_attribute(Person, :age) { |v| v >= 18 }
    @bob = Person.new
  end
  
  def test_refueses_invalid_values
    assert_raises RuntimeError, 'Invalid attribute' do
      @bob.age = 17
    end
  end
end
```

```ruby
def add_checked_attribute(klass, attribute, &validation)
  klass.class_eval do
    define_method "#{attribute}=" do |value|
      raise 'Invalid attribute' unless validation.call(value)
      instance_variable_set("@#{attribute}", value)
    end
    
    define_method attribute do
      instance_variable_get "@#{attribute}"
    end
  end
end
```

### 把内核方法改造成一个类宏，让他对所有类都可以用

```ruby
require 'test/unit'

class Person
	attr_checked :age do |v|
    v >= 18
  end
end

class TestCheckedAttribute < Test::Unit::TestCase
  def setup
    @bob = Person.new
  end
end
```

```ruby
class Class
  def attr_checked(attribute, &validation)
    define_method "#{attribute}=" do |value|
      raise 'Invalid attribute' unless validation.call(value)
      instance_variable_set("@#{attribute}", value)
    end
    define_method attribute do
      instance_variable_get "@#{attribute}"
    end
  end
end
```

### 写在一个模块中

```ruby
class Person
  include CheckedAttributes
  
  attr_checked :age do |v|
    v >= 18
  end
end
```

```ruby
module CheckedAttributes
  def self.included(base)
    base.extend ClassMethods
  end
  
  module ClassMethods
    def attr_checked(attribute, &validation)
      define_method "#{attribute}=" do |value|
        raise 'Invalid attribute' unless validation.call(value)
        instance_variable_set("@#{attribute}", value)
      end
      define_method attribute do
        instance_variable_get "@#{attribute}"
      end
    end
  end
end
```

