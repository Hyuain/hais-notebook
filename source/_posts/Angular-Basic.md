---
title: Angular：Basic
date: 2020-03-24 15:40:32
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, 框架, Angular]
---

抄一下 Angular 的文档。

<!-- more -->

# 基本概念

## Modules

Angular 里面有自己的模块化系统，也就是 NgModule，他与 ES6 的模块系统不同，又互为补充。每个 Angular 应用中都有一个 **根模块（AppModule）**，并且位于 `app.module.ts` 中，可以通过引导这个模块来启动整个应用

```typescript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
@NgModule({
  imports:      [ BrowserModule ], // 这个模块中的 component templates 需要使用到的模块
  providers:    [ Logger ],        // 这个模块中全局 services 的 creators
  declarations: [ AppComponent ],  // 包括了这个 NgModule 下的 components、directives、pipes
  exports:      [ AppComponent ],  // 包括了可能在其他模块使用的东西
  bootstrap:    [ AppComponent ]   // 应用的主视图，也就是 root component，只有根模块才有这个属性
})
export class AppModule { }
```

模块可以为其中的组件提供一个编译上下文环境（compilation context），根模块一定会有一个根视图

## Components

每个应用都至少一个组件，也就是 root component，Angular 的组件一般长这个样子：

```typescript
@Component({ // @Component 装饰器指出接下来的这个类是个组件类，并且指定元数据，组件的元数据将告诉 Angular 怎样将模板与组件关联起来并构成视图
  selector:    'app-hero-list', // 一个 CSS 选择器，告诉 Angular 在哪个标签插入组件实例
  templateUrl: './hero-list.component.html', // 该组件 HTML 模板文件相对于这个组件文件的地址
  providers:  [ HeroService ] // 当前组件所需的 services 的 providers
})
export class HeroListComponent implements OnInit {
/* . . . */
}
```

### 模板语法

Angular 采用形如这样的模板语法：

```html
<h2>Hero List</h2>

<p><i>Pick a hero from the list</i></p>
<ul>
  <li *ngFor="let hero of heroes" (click)="selectHero(hero)">
    {{hero.name}}
  </li>
</ul>

<app-hero-detail *ngIf="selectedHero" [hero]="selectedHero"></app-hero-detail>
```

`@Component` 元数据会告诉 Angular 模板从哪里找：
- 可以直接将模板写在 `@Component` 元数据的 `template` 中
- 可以将模板文件的地址告诉 `@Component` 元数据的 `template` 

```bash
# 默认将模板文件单独放置
ng generate component hero
# -t 是 inlineTemplate=true 的简写
ng generate component hero -t 
```

#### 数据绑定

与 Vue 比较类似，Angular 这样几种数据绑定方式：

- 从组件到 DOM `{{value}}` `[property]="value"` `bind-property="value"`
- 从 DOM 到组件 `(event)="handler"` `on-event="value"`
- 双向数据绑定 `[(ng-model)]="property"` `bindon-target="expression"`

![Angular 的数据绑定1](https://angular.cn/generated/images/guide/architecture/component-databinding.png)

在父子组件中也是如此：

![Angular 的数据绑定2](https://angular.cn/generated/images/guide/architecture/parent-child-binding.png)

其插值语法、表达式、`on`、`bind`、`bindon` 等于 Vue 类似

#### 管道

Angular 自带了一些管道，可以帮助我们将输入值转换为对应的输出值，`|` 被称为 **管道操作符**：

```html
<!-- Default format: output 'Jun 15, 2015'-->

<p>Today is {{today | date}}</p>

<!-- fullDate format: output 'Monday, June 15, 2015'-->

<p>The date is {{today | date:'fullDate'}}</p>

<!-- shortTime format: output '9:43 AM'-->

<p>The time is {{today | date:'shortTime'}}</p>
```

#### 指令

Angular 自带了一些指令，比如：

```html
<!-- 结构型指令 -->
<li *ngFor="let hero of heroes"></li>
<app-hero-detail *ngIf="selectedHero"></app-hero-detail>

<!-- 属性型指令 -->
<input [(ngModel)]="hero.name">
```

## Service and Dependency Injection

组件可以把诸如获取数据、验证用户输入、往控制台写日志等等委托给各种服务，可以在需要时将服务 **注入** 到组件中

```typescript
export class Logger {
  log(msg: any)   { console.log(msg); }
  error(msg: any) { console.error(msg); }
  warn(msg: any)  { console.warn(msg); }
}
```

```typescript
export class HeroService {
  private heroes: Hero[] = [];

  constructor(
    private backend: BackendService,
    private logger: Logger) { }

  getHeroes() {
    this.backend.getAll(Hero).then( (heroes: Hero[]) => {
      this.logger.log(`Fetched ${heroes.length} heroes.`);
      this.heroes.push(...heroes); // fill cache
    });
    return this.heroes;
  }
}
```
