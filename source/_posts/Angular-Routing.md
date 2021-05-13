---
title: Angular：Routing
date: 2020-03-25 09:10:06
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, Angular]
---

抄一下 Angular 的文档。

<!-- more -->

# AppRoutingModule

在 Angular 中，我们通过一个分离的、顶级的模块来用于导航，通常我们将其命名为 `AppRoutingModule`，他通常在 `src/app` 文件夹的 `app-routing.module.ts` 中。

```bash
ng generate module app-routing --flat --module=app
# --flat 是他不会再在 src/app 中新建其他的文件夹
# --module=app 则使得其被写在 AppModule 的 imports 中
```

通常我们需要将这个文件改造成这个样子：

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HeroesComponent } from './heroes/heroes.component';
import { DashboardComponent } from './dashboard/dashboard.component'
import { HeroDetailComponent } from './hero-detail/hero-detail.component'

const routes: Routes = [
  { path: '', redirectTo: '/dashboard', pathMatch: 'full' },
  { path: 'dashboard', component: DashboardComponent },
  { path: 'detail/:id', component: HeroDetailComponent },
  { path: 'heroes', component: HeroesComponent }
]

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  // 由于我们是在应用的 root level 定义路由，因此需要使用 forRoot
  // forRoot 方法提供了路由中需要的 service providers 和 directives
  exports: [RouterModule]
})
```

然后需要在 html 模板中这样写：

```html
<nav>
  <a routerLink="/dashboard">Dashboard</a>
  <a routerLink="/heroes">Heroes</a>
</nav>
<router-outlet></router-outlet>
```

我们可以通过 `+this.route.snapshot.paramMap.get('id')` 这样的方法来从当前的地址中获取到 params
