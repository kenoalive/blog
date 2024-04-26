---
title: NestJs新手入门
date: 2024-04-26 00:00:00
image: image.png
categories: ["后端"]
weight: 1
---

> 对于后端框架一直停留在用 express 和 koa 写写 curd 的水平，今天就学一下用 Nest 写写 curd 😂

## 项目启动

```bash
    npm i -g @nestjs/cli
    nest new nest-crud-demo

    cd nest-crud-demo
    yarn && yarn run start:dev
```

打开 localhost:3000，有内容说明运行成功

## 项目结构

```css
src
├── app.controller.spec.ts
├── app.controller.ts
├── app.module.ts
├── app.service.ts
├── main.ts
```

```ts
// src/app.module.ts
import { Module } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

`AppModule`是应用程序的根模块，根模块提供了用来启动应用的引导机制，可以包含很多功能模块。
`AppModule`是通过创建一个 `@Module()` 装饰器修饰的 TypeScript 类来定义的

- providers：业务逻辑模块
- AppController：处理 http 请求和路由

```ts
// app.controller.ts
import { Controller, Get } from "@nestjs/common";
import { AppService } from "./app.service";

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

使用`@Controller`装饰器来定义控制器, `@Get`是请求方法的装饰器，对`getHello`方法进行修饰， 表示这个方法会被 GET 请求调用。

```ts
// app.service.ts
import { Injectable } from "@nestjs/common";

@Injectable()
export class AppService {
  getHello(): string {
    return "Hello World!";
  }
}
```

@Injectable()用来标记可注入的服务

## Controller

> 官方文档：
> Controllers are responsible for handling incoming requests and returning responses to the client.
> **Controllers 负责处理传入请求并向客户端返回响应。**

`Controller` 的作用就是提供 api 路由、处理前端传入路由的 http 请求和返回响应

将代码里的 app.controller.ts 代码改为：

```ts
import { Controller, Get, Post } from "@nestjs/common";
import { AppService } from "./app.service";

@Controller("app")
export class AppController {
  constructor(private readonly appService: AppService) {}
  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
  @Get("users")
  findAll(): string {
    return "All users"; // [All User's Info] 暂时代替所有用户的信息
  }
  @Post("list/:id")
  update() {
    return "update";
  }
}
```

`@Controller('app')`这意味着所有在该控制器下定义的路由都将以 `/app` 作为路径前缀。

- `getHello()` 方法使用了 `@Get()` 装饰器来指定路由路径为 `/app`，它调用了 `AppService` 中的 `getHello()` 方法，并返回相应的字符串。
- `findAll()` 方法使用了 `@Get('users')` 装饰器来指定路由路径为 `/app/users`，它暂时返回了一个字符串表示所有用户的信息。通常情况下，我们会调用相应的服务方法

访问 `http://localhost:3000/app/users` 返回 all users

使用 postman 请求 `http://localhost:3000/app/list/111` 返回 update

## Provider

`provider`的含义就是提供服务，一般用来操作数据库以及处理数据

> 个人理解就是：module 用来包含 controller 和 service，module 本身可以导入和导出其他 module，
> controller 用来处理路由和请求，service 用来操作数据库

## nest 快捷命令

```bash
nest g [文件类型] [文件名] [文件目录]
```

现在新建一个完整的 post 模块
运行 `nest g resource post`
会在`/src/post/` 文件夹下生成 `post.controller.ts`、`post.module.ts`、`post.service.ts`

## Prisma + Mysql

> 主流的 Nest 的 ORM 应该是 TypeORM，但是这里想折腾一下 Prisma

### 安装依赖

```bash
yarn add prisma-binding @prisma/client
yarn add -D prisma
```

### 初始化

```bash
npx prisma init
```

执行以上命令会创建.env 文件与 prisma 文件夹

根据本机数据库修改`.env`
![baseUrl](Untitled.png)

修改`schema.prisma`

```ts
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

```

### 定义数据模型

编辑 `schema.prisma` 文件，定义实体、字段、关联等。例如：

```ts
model Product {
  id        String   @id @default(cuid())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  name        String
  description String?
  price       Decimal
  sku         String  @unique
  published   Boolean @default(false)
}
```

运行 `npx prisma migrate dev` 修改数据库，使其保持一致

这个 Prisma schema 同步数据库的过程，就被称之为  migration。每次迁移，都会生成一个迁移文件，存放在  prisma/migrations 下。

手动在数据库里更改数据库结构后，需要运行 `npx prisma db pull` 将数据库字段同步到`schema.prisma`

### 配置 tsconfig.json

```json
{
  "paths": {
    "@prisma/client": ["./prisma/client"]
  }
}
```

### 创建 PrismaClient 模块和服务

使用 nest 命令创建：

```bash
nest g mo prisma
nest g s prisma
```

将该 PrismaModule 设置成全局，并导出 PrismaService：

```ts
// prisma.service.ts
import { PrismaClient } from "@prisma/client";
import { Injectable } from "@nestjs/common";

@Injectable()
export class PrismaService extends PrismaClient {
  constructor() {
    super();
  }
}
```

```ts
// prisma.module.ts
import { Global, Module } from "@nestjs/common";
import { PrismaService } from "./prisma.service";

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

### REST API

使用 nest 生成 products 的 REST API：
`nest g res products`

修改生成的`products.service.ts`

```ts
...
import { Injectable } from '@nestjs/common';
import { PrismaService } from 'src/prisma/prisma.service';

@Injectable()
export class ProductsService {
  constructor(private readonly prismaService: PrismaService) {}
  findAll() {
    console.log('findAll');
    return this.prismaService.product.findMany();
  }
}

```

修改 `products.controllers.ts`

```ts
import { Controller, Get } from "@nestjs/common";
import { ProductsService } from "./products.service";

@Controller("products")
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}
  @Get()
  findAll() {
    return this.productsService.findAll();
  }
}
```

访问 http://localhost:3000/products，成功返回空数组
