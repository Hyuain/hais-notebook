---
title: Angular：HttpClient
date: 2020-03-25 09:30:26
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, 框架, Angular]
---

HttpClient 是 Angular 用来与服务器通过 HTTP 交流的装置。

<!-- more -->

# 引入 HttpClient

可以在 `app.module.ts` 中引入

```typescript
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    HttpClientModule,
  ],
})
```

然后在需要使用的 service 中注入

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http'

@Injectable({
  providedIn: 'root'
})
export class HeroService {
  constructor(
    private http: HttpClient 
  ) {}
}
```

# 基本使用

## 获取数据

```typescript
getHeroes(): Observable<Hero[]> {
  return this.http.get<Hero[]>('/someurl') // 他会返回一个 Observable 对象，里面具体的类型是 Hero[]
}
```

