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

数据库需要在 `/config/database.yml` 中进行配置。

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

路由需要在 `/config/routes.rb` 中进行配置。

```ruby
Rails.application.routes.draw do
  get '/hello', to: 'first#hello' # get 请求路径 /hello 的时候，会去找 FirstController 上的 hello 方法 
  get '/hi', to: 'first#hi'
end
```

## Controller

controller 需要在 `/app/controllers` 中进行配置。
比如如上面 routes 中的配置所说，我们需要一个 `/app/controllers/first_controller.rb` 文件，里面有一个 `FirstController` class，其中对应有 `hello` 和 `hi` 方法：

```ruby
# 继承自 ApplicationController
class FirstController < ApplicationController
  def hello
    render plain: 'hello'
  end
end
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

一般前后端分离的情况下，不是由 rails 来渲染 HTML，当然 rails 也可以用于渲染 HTML，这时我们需要建立一个新的文件 `/app/views/first/hello.html`，然后在 FirstController 中这样写：

```ruby
class FirstController < ApplicationController
  def hello
    render 'first/hello'
  end
end
```

注意，如果使用 API 模式，默认是不支持渲染 html 的，我们需要在 `/app/controllers/application_controller.rb` 中进行引入：

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

此外很多 html 有公共的部分，我们可以将其放在 `views/layouts/application.html.erb` 中，然后在其中空出类似插槽的东西：

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