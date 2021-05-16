---
title: TypeORM
date: 2021-04-11 18:51:00
tags:
  - 入门
categories:
  - [全栈, NodeJS]
---

TypeORM 是一个 Node 中对 TypeScript 支持比较好的关系对象映射，支持关联、事务、数据库迁移，同类产品还有 Sequelize。
Mac 和 Linux 可以用 & 同时执行两个命令，Windows 可以用 concurrently 这个 npm 库。

<!-- more -->

# 启动 Postgresql 数据库

在项目目录里面新增 `blog-data` 目录，并将其添加进 `.gitignore` 中。
 
然后启动 postgresql：

```bash
docker run -v "$PWD/blog-data":/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_USER=blog -e POSTGRES_HOST_AUTH_METHOD=trust -d postgres
```

将上述命令的 `-e POSTGRES_HOST_AUTH_METHOD=trust` 替换成 `-e POSTGRES_PASSWORD=123456` 就可以设置密码

然后可以这样进入 postgresql：

```bash
docker exec -it id bash
psql -U username -W
```

然后可以执行 psql 的命令：

```bash
\l # list databases
\c # connect to a databases
\dt # display tables
```

# 创建 database

由于 TypeORM 没有为我们提供单纯创建数据库的 API，我们需要进入数据库用 SQL 语句来进行创建：

```postgresql
CREATE DATABASE blog_production ENCODING 'UTF8' LC_COLLATE 'en_US.utf8' LC_CTYPE 'en_US.utf8';
```

一般来说需要创建三个数据库：development、test、production

# 安装 TypeORM

[官方文档在这里](https://typeorm.io/#/)。

注意！当按照官方文档操作，运行命令 `typeorm init --database postgres` 之后，`.gitignore` 、 `package.json` 和 `tsconfig.json` 将会被更改，这时需要检查他的更改并酌情处理！

比如当我们在使用 Next 搭配 TypeORM 的时候，会遇到问题是 Next 使用 Babel 处理 TypeScript，而 TypeORM 则推荐使用 ts-node，这样就冲突了：
1. 先从 `package.json` 中删除命令擅自添加的 `ts-node`
2. 再想办法运行 `src/index.ts`
   
## 自行使用 babel 来进行编译，然后再用 node 运行编译好的 js

TypeORM 想要我们直接运行 `src/index.ts`，但是我们需要安装 `@babel/cli` 之后使用命令 `npx babel ./src --out-dir dist --extensions ".ts,.tsx"` 来运行。

这时可能会遇到报错 `Support for the experimental syntax 'decorators-legacy' isn't currently enabled`。

解决方案可以看[这篇文章](https://stackoverflow.com/questions/52262084/syntax-error-support-for-the-experimental-syntax-decorators-legacy-isnt-cur)。
同时可以参考[next 文档](https://nextjs.org/docs/advanced-features/customizing-babel-config)来配置 `.babelrc`

然后我们就可以在 dist 里面找到编译好的 `index.js`，用 node 直接运行他，但是注意需要修改 `ormconfig.json` 里面的配置，尤其是这一部分：

```json
{
  "entities": [
    "dist/entity/**/*.js"
  ]
}
```

## 让 Next 帮我们编译，帮我们运行

我们可以将上面 TypeORM 生成的 `src/index.ts` 放到 `pages` 目录下，这样 Next 就会在 请求(开发)/打包(生产) 的时候帮我们编译并执行。同样需要注意修改 `ormconfig.json`。

# synchronize 功能

同步功能可以在 `ormconfig.json` 里面的 `synchronize` 项进行配置。

如果打开 synchronize，每次 `createConnection` 的时候都会将 entity 里面的表同步到我们的数据库中，这样就很可能导致我们在修改 User 的时候把数据删掉，这在生产环境肯定是不允许的。因此我们可以在一开始就把这个功能给关掉。

# 通过 migration 创建表

```bash
npx typeorm migration:create -n CreatePost
```

然后对其创建的 `/src/migration/[timestamp]-CreatePosts.ts` 进行修改：

```typescript
import { MigrationInterface, QueryRunner, Table } from "typeorm"

export class CreatePosts1620523637192 implements MigrationInterface {

  public async up(queryRunner: QueryRunner): Promise<void> {
    // 升级数据库
    return await queryRunner.createTable(new Table({
      name: "posts",
      columns: [
        { name: "id", type: "int", isPrimary: true, isGenerated: true, generationStrategy: "increment" },
        { name: "title", type: "varchar" },
        { name: "content", type: "text" },
      ],
    }))
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    // 降级数据库
    return await queryRunner.dropTable("posts")
  }

}
 
```

然后同样的，我们需要将其编译成 js，然后配置 `ormconfig.json` 中的 `migrations` 路径，然后再执行：

```bash
npx typeorm migration:run
```

他会执行 `up` 中的代码，于是就创建了一个表。

当发生错误需要撤销这次 migration 的时候，执行：

```bash
npx typeorm migration:revert
```

他会执行 `down` 中的代码，在这里就会删除这个表。

# 将数据映射到实体从而操作他

在数据库中创建一个表之后，如果想操作他，我们通常需要他映射到 实体(Entity) 上，可以通过这个命令达成：

```bash
npx typeorm entity:create -n Post
```

需要将 Entity Post 对应上数据库里面的信息：

```typescript
import { Column, Entity, PrimaryGeneratedColumn } from "typeorm"

@Entity("posts")
export class Post {

  @PrimaryGeneratedColumn("increment")
  id: number

  @Column("varchar")
  title: string

  @Column("text")
  content: string
}
```

可以使用 [EntityManager](https://typeorm.io/#/entity-manager-api) 和 [Repository](https://typeorm.io/#/repository-api) 这两种使用实体的方式，他们只是封装思路不同而已。

# Seed

也叫数据填充，我们可以通过 seed 脚本来构造数据。

# 创建数据表关联

可以参考 [这篇文档](https://typeorm.io/#/relations)

```typescript
@Entity()
export class User {
  //... 其他字段
  
  @OneToMany(type => Post, post => post.author)
  posts: Post[]
}
```