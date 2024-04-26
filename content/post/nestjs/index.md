---
title: NestJs入门
date: 2024-04-26 10:00:00
image: image.png
categories: ["后端"]
weight: 2 
---
> 对于后端框架一直停留在用express和koa写写curd的水平，今天就学一下用Nest写写curd 😂

### 项目启动
```bash
    npm i -g @nestjs/cli
    nest new nest-crud-demo  

    cd nest-crud-demo
    yarn && yarn run start:dev
```
打开localhost:3000，有内容说明运行成功

### 项目结构

``` css
src
├── app.controller.spec.ts
├── app.controller.ts
├── app.module.ts
├── app.service.ts
├── main.ts
```

```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
`AppModule`是应用程序的根模块，根模块提供了用来启动应用的引导机制，可以包含很多功能模块。
`AppModule`是通过创建一个 `@Module()` 装饰器修饰的TypeScript类来定义的
- providers：业务逻辑模块
- AppController：处理http请求和路由
```ts
// app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}

```
使用`@Controller`装饰器来定义控制器, `@Get`是请求方法的装饰器，对`getHello`方法进行修饰， 表示这个方法会被GET请求调用。

```ts
// app.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

@Injectable()用来标记可注入的服务

### Controller
`Controller` 的作用就是提供api路由、处理前端传入路由的http请求和返回响应
将代码里的app.controller.ts代码改为：
```ts
import { Controller, Get, Post } from '@nestjs/common';
import { AppService } from './app.service';

@Controller('app')
export class AppController {
  constructor(private readonly appService: AppService) {}
  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
  @Get('users')
  findAll(): string {
    return "All users"; // [All User's Info] 暂时代替所有用户的信息
  }
  @Post("list/:id")
  update(){ return "update"}
}
```
`@Controller('app')`这意味着所有在该控制器下定义的路由都将以 `/app` 作为路径前缀。
 - `getHello()` 方法使用了 `@Get()` 装饰器来指定路由路径为 `/app`，它调用了 `AppService` 中的 `getHello()` 方法，并返回相应的字符串。
 - `findAll()` 方法使用了 `@Get('users')` 装饰器来指定路由路径为 `/app/users`，它暂时返回了一个字符串表示所有用户的信息。通常情况下，我们会调用相应的服务方法

 访问 `http://localhost:3000/app/users` ，返回all users
 使用postman请求 `http://localhost:3000/app/list/111`返回 update



 ### Provider
`provider`的含义就是提供服务，一般用来操作数据库以及处理数据

> 个人理解就是：module用来包含controller和service，module本身可以导入和导出其他module
controller用来处理路由和请求，service用来操作数据库


### nest快捷命令
```bash 
nest g [文件类型] [文件名] [文件目录]
```
现在新建一个完整的post模块
运行 `nest g resource post`
会在`/src/post/` 文件夹下生成 `post.controller.ts`、`post.module.ts`、`post.service.ts`