---
title: Rails 的项目搭建
date: 2021-02-13 11:48:41
tags:
  - 入门
categories:
  - [全栈, Ruby, Rails]
---

Rails 的一个项目是如何搭建起来的。

<!-- more -->

# Gem 和 Bundle

类似于 npm，gem 用于全局安装依赖，bundle 用于局部安装依赖。

如果 `gem install` 安装很慢，按照 [RubyChina](https://gems.ruby-china.com/) 的方法进行操作。

# 创建 Rails 项目

- `gem install rails`
-
```bash
rails new [ProjectName] --database=postgresql --skip-action-mailbox --skip-action-text --skip-sprockets --skip-javascript --skip-turbolinks --skip-system-test --skip-test --api --skip-webpack-install
```

然后使用 `bin/rails server` 命令启动服务器，但是这时会报错，因为我们还没有数据库。

# 创建 docker 容器

```bash
docker run -v [容器路径]:/var/lib/postgresql/data -p 5001:5432 -e POSTGRES_USER=[用户名] -e POSTGRES_PASSWORD=[密码] -d postgres:[版本号 也可以不指定]
```

如果速度太慢，需要使用镜像，按照 [此教程](http://guide.daocloud.io/dcs/daocloud-9153151.html) 进行配置。
可将镜像地址配置为中科大镜像地址：https://docker.mirrors.ustc.edu.cn

## 其他的 docker 命令

- `docker ps -a` 查看所有的容器（Containers）
- `docker kill [id]` 关闭对应的容器
- `docker restart [id]` 重启关闭的容器
- `docer rm [id]` 删除对应的容器
- `docker container prune` 删除无用的容器，以节省空间

# 配置数据库

数据库需要在 `config/database.yml` 中进行配置。

```yaml
default: &default # 默认公共配置
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  database:
  username:
  password:
  host: localhost
  port: 5001

development:
  <<: *default
  database: xxx_development

test:
  <<: *default
  database: xxx_test

production:
  <<: *default
  database: morney_rails_1_production
  username: morney_rails_1
  password: <%= ENV['MORNEY_RAILS_1_DATABASE_PASSWORD'] %>

```

# 创建数据库

```bash
bin/rials db:create
```

# Hello Rails

## Routes

路由需要在 `config/routes.rb` 中进行配置。

```ruby
Rails.application.routes.draw do
  get '/hello', to: 'first#hello' # get 请求路径 /hello 的时候，会去找 FirstController 上的 hello 方法 
  get '/hi', to: 'first#hi'
end
```

可以通过 `bin/rails routes` 命令查看所有的路由。

## Controller

### 手动创建 controller

controller 需要在 `app/controllers` 中进行配置。
比如如上面 routes 中的配置所说，我们需要一个 `app/controllers/first_controller.rb` 文件，里面有一个 `FirstController` class，其中对应有 `hello` 和 `hi` 方法：

```ruby
# 继承自 ApplicationController
class FirstController < ApplicationController
  def hello
    render plain: 'hello'
  end
end
```

### 通过命令创建 controller

```bash
bin/rails g controller users
``` 

### 渲染 JSON

```ruby
class FirstController < ApplicationController
  def hello
    render json: { name: 'harvey' }
  end
end
```

## View

### 渲染 HTML

一般前后端分离的情况下，不是由 rails 来渲染 HTML，当然 rails 也可以用于渲染 HTML，这时我们需要建立一个新的文件 `app/views/first/hello.html`，然后在 FirstController 中这样写：

```ruby
class FirstController < ApplicationController
  def hello
    render 'first/hello'
  end
end
```

注意，如果使用 API 模式，默认是不支持渲染 html 的，我们需要在 `app/controllersapplication_controller.rb` 中进行引入：

```ruby
class ApplicationController < ActionController::API
  include ActionView::Layouts
end
``` 

### Layout

erb 可以使用 `<% %>` 来包裹语句：

```html
<% a = 5 %>
<% if a < 10 %>
  a 小于 10
<% end %>
```

也可以使用 `<%= %>` 来表示需要渲染的内容：

```html
<%= @dataInController %>
```

此外很多 html 有公共的部分，我们可以将其放在 `views/layoutsapplication.html.erb` 中，然后在其中空出类似插槽的东西：

```html
<!-- 前略 -->
<%= yield %>

<% if content_for? :footer %>
  <footer>
    <%= yield :footer %>
  </footer>
<% end %>
<!-- 后略 -->
```

```html
<p>普通内容</p>
<% content_for :footer do %>
  页脚内容
<% end %>
```

# User

### 第一步：创建 Model 与数据表

```bash
bin/rails g model User
# 也可以像下面这样写，他会在数据库迁移脚本中多两个字段
bin/rails g model User email:string password_digest:string
```

执行此命令后，会创建一个数据库迁移脚本，和一个新文件 `app/model/user.rb`。

数据库迁移脚本形式如下：

```ruby
class CreateUsers < ActiveRecord::Migration[6.1]
  def change
    create_table :users do |t|
      t.string :email
      t.string :password_digest

      t.timestamps
    end
  end
end
```

然后需要运行迁移脚本，创建 User 表：

```bash
bin/rails db:migrate
```

然后可以在 rubyMine 中查看数据库、User 表。

### 第二步：配置路由

```ruby
Rails.application.routes.draw do
  get '/users', to: 'users#index'
  get '/users/:id', to: 'users#show'
  post '/users', to: 'users#create'
  delete '/users/:id', to: 'users#destroy'
  patch '/users/:id', to: 'users#update'
end
```

简单的增删改查我们不需要配置如上的路由，只需要下面这一句，他就会帮我们自动创建路由：

```ruby
resources :users
```

### 第三步：在 Rails Console 中尝试进行增删改查

可以在 Rails Console 中先体验一下增删改查是如何进行的：

```bash
bin/rails console
```

然后执行：

```ruby
u = User.new
u.email = '1@qq.com'
u.password_digest = '123456'
u.save
```

就会自动执行一个 SQL 语句，**创建** 一个新用户。

可以通过 `User.first` `User.second` 来 **查看** 已有的用户。

输入 `exit` 或者按 `Ctrl`+`D` 可以退出 Rails Console。

### 第四步：存储密码

我们平时的 User 表中不会存储密码的明文，而是存储一个 `password_digest` 字段，这可以通过 has_secure_password 来实现。

放开 `Gemfile` 中 `bcrypt` 的注释，通过 `bundle install` 安装依赖。

然后需要在 `app/modles/user.rb` 中加上 `has_secure_password`：

```ruby
class User < ApplicationRecord
  has_secure_password
end
```

这时在 Rails Console 中执行以下代码试试：

```ruby
u = User.new
u.email = '1@qq.com'
u.password = '123456'
u.password_confirmation = '123456'
u.save
```

这样就会自动在表中 `password_digest` 中存一段密文。

然后可以通过这样来比对用户的密码是否正确：

```ruby
u.authenticate('123')
```

### 第五步：创建与配置 Controller

我们可以通过如下命令创建 Controller，得到 `app/controllers/users_controller.rb`：

```bash
bin/rails g controller users
``` 

```ruby
class UsersController < ApplicationController
  def create
    def create
      user = User.new({ email: params[:email], password: params[:password], password_confirmation: params[:password_confirmation] })
      if user.save # 会返回保存是否成功，将成功与失败分别返回给前端
        render json: { resource: user }, status: 200
      else
        render json: { errors: user.errors }, status: 400
      end
    end
  end
end
```

#### 模拟请求进行调试

##### 使用 RubyMine 中的 HTTP Client

`Double Shift` - `HTTP Client`，然后可以通过 examples 看看如何使用。

##### 使用 Postman 测试



### 第六步：在 Model 中进行数据校验

```ruby
class User < ApplicationRecord
  has_secure_password

  validates_presence_of :email # email 必须存在
  validates_presence_of :password # password 必须存在
  validates_presence_of :password_confirmation, on: [:create] # 在创建时，确认输入密码必须存在

  validates_format_of :email, with: /.+@.+/, if: :email # 在 email 存在时校验邮箱格式
  validates_length_of :password, minimum: 6, on: [:create], if: :password # 在创建时，校验密码格式
end
```

### 第七步：错误信息汉化

首先需要在 `config/initializers/local.rb` 中添加：

```ruby
I18n.available_locales = [:en, 'zh-CN']
I18n.default_locale = 'zh-CN'
```

然后按照提示在 `config/locales/zh-CN.yml` 中进行配置：

```yaml
zh-CN:
  activerecord:
    errors:
      models:
        user:
          attributes:
            password:
              blank: 密码不能为空
              too_short: 密码不能少于6个字符
            email:
              blank: 邮箱不能为空
              invalid: 邮箱必须含有 @ 字符
            password_confirmation:
              blank: 请添加确认密码
              confirmation: 两次密码不匹配
```