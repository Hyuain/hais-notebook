---
title: Rails
date: 2023-01-16 11:48:41
categories:
  - - 全栈
tags:
  - RB1
---

Rails 的一个项目是如何搭建起来的。

<!-- more -->

# Gem 和 Bundle

类似于 npm，gem 用于全局安装依赖，bundle 用于局部安装依赖。

如果 `gem install` 安装很慢，按照 [RubyChina](https://gems.ruby-china.com/) 的方法进行操作。

# Development

## 创建 Rails 项目

```bash
gem install rails
rails new [ProjectName] --database=postgresql --skip-action-mailbox --skip-action-text --skip-sprockets --skip-javascript --skip-turbolinks --skip-system-test --skip-test --api --skip-webpack-install
```

然后使用 `bin/rails server` 命令启动服务器，但是这时会报错，因为我们还没有数据库。

## 创建 docker 容器

```bash
docker run -v [容器路径]:/var/lib/postgresql/data -p 5001:5432 -e POSTGRES_USER=[用户名] -e POSTGRES_PASSWORD=[密码] -d postgres:[版本号 也可以不指定]
```

如果速度太慢，需要使用镜像，按照 [此教程](http://guide.daocloud.io/dcs/daocloud-9153151.html) 进行配置。
可将镜像地址配置为中科大镜像地址：https://docker.mirrors.ustc.edu.cn

### 其他的 docker 命令

- `docker ps -a` 查看所有的容器（Containers）
- `docker kill <id|name>` 关闭对应的容器
- `docker restart <id|name>` 重启关闭的容器
- `docker rm <id|name>` 删除对应的容器
- `docker container prune` 删除无用的容器，以节省空间

## 准备数据库

### 配置数据库

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

### 创建数据库

```bash
bin/rials db:create
```

### 查看数据库

```bash
# 进入虚拟机并运行 bash：
docker exec -it <容器名> bash
# 登录
psql -U <用户名>
# 连接数据库
\c <数据库名称>
# display tables
\dt
# 查看某一个表的内容
select * from <表名称> limit 10;
```

## Hello Rails

### Routes

路由需要在 `config/routes.rb` 中进行配置。

```ruby
Rails.application.routes.draw do
  get '/hello', to: 'first#hello' # get 请求路径 /hello 的时候，会去找 FirstController 上的 hello 方法 
  get '/hi', to: 'first#hi'
end
```

可以通过 `bin/rails routes` 命令查看所有的路由。

### Controller

#### 手动创建 controller

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

#### 通过命令创建 controller

```bash
bin/rails g controller users
```

#### 渲染 JSON

```ruby
class FirstController < ApplicationController
  def hello
    render json: { name: 'harvey' }
  end
end
```

### View

#### 渲染 HTML

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

#### Layout

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

## 注册功能

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
    user = User.new create_params
    user.save
    # new + save 可以简写为 User.create
    render_resource user
  end

  def create_params
    params.permit(:email, :password, :password_confirmation)
  end

  def render_resource(resource)
    # 也可以用 resource.valid?，不过据说会触发重新校验
    if resource.errors.empty?
      render json: { resource: resource }, status: 200
    else
      render json: { errors: resource.errors }, status: 400
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

### 第八步：发送邮件

可以通过 `mailer` 来发送邮件，点击 [这里查看官方文档](https://ruby-china.github.io/rails-guides/action_mailer_basics.html)。

```bash
bin/rails generate mailer UserMailer
```

这个命令会创建文件 `app/mailers/user_mailer`，详细请看 [相关 commit](https://github.com/Hyuain/ruby-demo/commits/master)

通过 `dotenv-rails` 和 `.env` `.env.local` 文件来抽出环境变量，注意 `.env.local` 需要被加入 `.gitignore` 中，防止将密码提交到 Git 记录中。

## 登录功能

### 第一步：配置路由

按照惯例我们需要先配置路由：

```ruby
Rails.application.routes.draw do
  resources :sessions, only: %i[create destroy]
end
```

### 第二步：创建 Model

Session 不需要写入数据库，故而不需要创建一个完整的 Model（继承于 ActiveRecord），而是一个轻量的 Model（继承于/include ActiveModel）。

```ruby
class Session
  include ActiveModel::Model
  attr_accessor :email, :password

  validates :email, presence: true
  validates :password, presence: true

  validates_format_of :email, with: /.+@.+/, if: :email
  validates_length_of :password, minimum: 6, if: :password
end
```

#### attr_accessor

```ruby
attr_accessor :xxx
```

1. 会声明一个对象的属性：`@xxx`
2. 会定义一个方法：`xxx`，用于获取 `@xxx` 的值
3. 会定义一个方法：`xxx=`，用于给 `@xxx` 赋值

### 第三步：创建 Controller

```bash
bin/rails g controller sessions
```

```ruby
class SessionsController < ApplicationController
  def create
    s = Session.new create_params
    s.validate
    render_resource s
  end

  def destroy

  end

  def create_params
    params.permit(:email, :password)
  end
end
```

### 第四步：自定义校验，校验账号密码

```ruby
class Session
  include ActiveModel::Model
  attr_accessor :email, :password, :email

  validate :check_email, if: :email
  validate :check_email_password_matched, if: Proc.new{ |s| s.email.present? and s.password.present? }

  def check_email
    @user ||= User.find_by email: email
    if user.nil?
      errors.add :email, :unregistered
    end
  end

  def check_email_password_matched
    @user ||= User.find_by email: email
    if user and !user.authenticate(password)
      errors.add :password, :mismatch
    end
  end
end
```

### 第五步：使用中间件，记录 session（cookie）

`config/application.rb`

```ruby
config.session_store :cookie_store, key: 'rails_demo_session_id'
config.middleware.use ActionDispatch::Cookies
config.middleware.use config.session_store, config.session_options
```

`session_controller.rb`

```ruby
class SessionsController < ApplicationController
  def create
    s = Session.new create_params
    s.validate
    render_resource s
    session[:current_user_id] = s.user.id # 将 id 记录起来
  end
end
```

这样前端在发送请求之后将会得到一个 cookie，RubyMine 可以在 `http-client.cookies` 中查看

### 第六步：获取当前用户的信息

```ruby
def current_user
  # 如果找不到，会返回 nil，ruby 这样会默认返回 current_user
  @current_user ||= User.find_by_id session[:current_user_id]
  # 如果找不到，会报错
  # User.find user_id
end
```

### 第七步：注销登录

```ruby
class SessionsController < ApplicationController
  def destroy
    session[:current_user_id] = nil
    head :ok
  end
end
```

将 session 中对应 id 的值删掉即可。

### 单元测试

使用 RSpec 进行单元测试，详情查看 [RSpec For Rails 文档](https://github.com/rspec/rspec-rails)。

详细请看 [相关 commit](https://github.com/Hyuain/ruby-demo/commits/master)

- Model 需要测试：validation 和 public 方法
- Controller 需要测试：响应体和响应头（status、body、cookie），一般要测试正反两种情况（成功和不成功）

### 自动生成文档

使用 `rspec_api_documentation` 可以直接通过单元测试生成 API 文档，详细内容可以 [查看这里](https://github.com/zipmark/rspec_api_documentation)

```bash
rake docs:generate
start doc/api/index.html
```

### 其他技巧

1. 可以在 `rspec_helper.rb` 中定义一些常用的工具函数
2. 可以使用 `rspec_api_documentation` 直接生成 API 文档

## 后端分页

使用 `kaminari` gem

## 使用中间表关联两个表

比如我们需要建立 tagging 来关联 record 和 tag

### 创建 Model

```bash
bin/rails g model tagging
```

```ruby
class CreateTaggings < ActiveRecord::Migration[6.1]
  def change
    create_table :taggings do |t|
      t.references :tag, null: false
      t.references :record, null: false

      t.timestamps
    end
  end
end
```

由于我们加了 `reference`，可以看到在 `schema.rb` 中多了一些东西：

```ruby
create_table "taggings", force: :cascade do |t|
  t.bigint "tag_id", null: false # 多了这一行
  t.bigint "record_id", null: false # 多了这一行
  t.datetime "created_at", precision: 6, null: false
  t.datetime "updated_at", precision: 6, null: false
  t.index ["record_id"], name: "index_taggings_on_record_id" # 多了这一行
  t.index ["tag_id"], name: "index_taggings_on_tag_id" # 多了这一行
end
```

然后需要调整 `model/tagging.rb` `model/record.rb` `model/tag.rb`

```ruby
class Tagging < ApplicationRecord
  belongs_to :record
  belongs_to :tag
end
```
```ruby
class Tag < ApplicationRecord
  has_many :taggings
  has_many :records, through: :taggings
end
```
```ruby
class Record < ApplicationRecord
  has_many :taggings
  has_many :tags, through: :taggings
end
```

## 使用 Factory Bot 加速测试

可以使用 [factory-bot](https://github.com/thoughtbot/factory_bot) 来帮助我们自动化一些流程，比如：

```ruby
# spec/factories/user_factory.rb
FactoryBot.define do
  factory :user do
    email { "test_email@qq.com" }
    password { "123456" }
    password_confirmation { "123456" }
  end
end
```
```ruby
# spec/requests/record.rb
user = create(:user)
```

# Deployment

## 购买并登录阿里云服务器

1. 去阿里云选择 ECS 服务器，购买一个。登录凭证选择使用密钥对。
2. 使用 `ssh root@公网ip` 登录到机器上
1. 公网 IP 很难记，建议记到 hosts 文件里

## 创建低权限用户

1. 进入 root 用户
2. 安装 git：`apt update; apt install git -y`
3. 创建 harvey 用户：`adduser harvey`，密码为 `123456`
4. 让 harvey 用户也能 ssh 登录
1. `mkdir -p /home/harvey/.ssh`
2. `cp ~/.ssh/authorized_keys /home/harvey/.ssh/`
3. `chown -R harvey:harvey /home/harvey/.ssh`
5. 退出 root（使用 `exit` 或 `ctrl + D`）
6. 现在就可以使用 `ssh root@harvey` 登录 harvey 用户了

## 安装 Docker

1. 进入 root 用户
2. 根据 [菜鸟教程](https://www.runoob.com/docker/ubuntu-docker-install.html) 来安装 Docker
3. 将 harvey 添加到 docker 用户组：`usermod -aG docker harvey`，这样 harvey 才能运行 docker 命令
4. 运行 `docker run hello-world`
5. 如果 `docker run hello-wrold` 运行很慢，可以参考 [这篇文章](https://developer.aliyun.com/article/29941) ，在 [这里](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors) 配置镜像加速器。

## 用 Docker 启动数据库

1. 进入 harvey 用户
2. 创建数据库目录 `mkdir /home/harvey/data`
3. 开启数据库 `docker run --net=host -v /home/harvey/data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=harvey -e POSTGRES_PASSWORD=123456 --name psql1 -d postgres`
1. 注意如果 `-net=host` 没有写的话会导致两个容器网不通
2. 此处用了端口 5432 而不是 5001，因为 `-net=host` 不支持 `5001:5432`
4. 测试数据库
1. `docker exec -t psql1 bash` 进入容器
2. `psql -U harvey`
3. `\l` 列出所有可用数据库

## 测试 Ruby 容器

1. 进入 harvey 用户
2. 创建测试文件 `echo "p 'Hi, I am Harvey'" > test.rb`
3. 使用 ruby 容器运行测试文件：`docker run -it --rm -v "$PWD":/usr/src/myapp -w /usr/src/myapp ruby ruby test.rb`
4. 看到 `Hi, I am Harvey` 就说明测试成功了，然后用 `rm test.rb` 删除测试代码

## 本地构建 Rails 镜像

1. 创建 `Dockerfile`：
```dockerfile
# 从哪里下载镜像，可以指定目录
FROM ruby

# 工作目录
WORKDIR /usr/src/app
# 把源代码拷贝到这个镜像中
COPY Gemfile .
COPY Gemfile.lock .
# 在镜像中运行
RUN gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
RUN gem install bundler
RUN bundle config mirror.https://rubygems.org https://gems.ruby-china.com
RUN bundle install
# 将所有的文件都拷贝到这个镜像中
COPY . .
# 更新 bin 目录
RUN bundle exec rake app:update:bin
# 暴露 3000 短开口
EXPOSE 3000
# 入口命令
CMD [ "bin/rails", "server", "-b","0.0.0.0", "-p","3000"]
```
2. 将 `config/database.yml` 复制到 `config/database.sample.yml`，再把原来的 `config/database.yml` ignore 掉
3. `docker build -t harvey/rails-demo:0.1 .`，得到镜像 `harvey/rails-demo` 版本为 0.1
3. 运行刚刚得到的镜像：`docker run --net=host -p 3000:3000 harvey/rails-demo:0.1`

