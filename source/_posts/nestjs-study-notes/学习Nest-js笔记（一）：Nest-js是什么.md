---
title: 学习Nest.js笔记（一）：Nest.js是什么
date: 2024-01-31 15:47:09
tags:
  - Nest.js
  - TypeScript
  - Node.js
  - 后端
categories:
  - [Nest.js学习笔记]
---

# 一、Nest.js 简介

Nest.js 是一个用于构建高效、可靠和可扩展的服务器端应用程序的框架。它 **完全使用 TypeScript 编写** ，但同时也 **兼容纯 JavaScript** 。Nest.js 结合了 **OOP（面向对象编程）** 、 **FP（函数式编程）** 和 **FRP（函数响应式编程）** 的特点。

其完全支持 TypeScript，并且结合了 **面向切面（AOP）** 的编程方式。

Nest.js 具有 **Spring MVC** 的风格，其中有 **依赖注入** 、 **控制反转** 等；而这些都是 **借鉴了 Angualr** 的成果。

先来看看在 Angular 当中，依赖注入（DI）和控制反转（IoC）是怎么体现的：

**依赖注入**

- 在 Angular 中，依赖注入是核心概念之一。它允许将依赖项（比如服务或对象）动态地提供给组件或服务，而不是在组件内部硬编码创建它们。
- 这种方法的好处是代码更加模块化，更易于测试和维护。
- Angular 通过其 DI 框架来实现这一点，主要是通过“注入器”（injector）和“提供者”（providers）来完成。注入器负责创建依赖项并将它们注入到组件或服务中。

**控制反转**

- 控制反转是一种设计原则，用于将组件的创建和管理控制权从组件自身转移到外部框架。
- 在 Angular 中，这主要通过依赖注入来实现。Angular 框架负责创建和管理服务或对象的实例，组件只需声明其依赖关系，框架则负责提供这些依赖项。

然后，我们可以了解一下 Nest.js 当中两者的体现方式：

**依赖注入**

- Nest.js 使用类似于 Angular 的依赖注入系统。在 Nest.js 中，你可以定义服务（service），然后在需要的地方（如控制器（controller））通过构造函数注入这些服务。
- 这种方法使得 Nest.js 应用中的组件之间的耦合度降低，提高了代码的可测试性和可维护性。

**控制反转**

- 与 Angular 类似，Nest.js 的控制反转是通过其依赖注入系统实现的。Nest.js 框架负责管理对象的生命周期和依赖关系，开发者只需关注业务逻辑。

Nest.js 的底层代码使用了 Express 和 Fastify，并在他们的基础上提供了一定程度的抽象，同时也将其 API 直接暴露给开发人员。这样可以轻松使用每个平台的无数第三方模块。

# 二、Nest.js 内置的两大框架：Express + Fastify

## Express

### 简介

[express 官网](https://expressjs.com/)

![image-20231201174223008](https://raw.githubusercontent.com/nonhana/PicGo-Pictures-Store/master/images/image-20231201174223008.png)

我相信每一个使用过 Node.js 自己写过后端的同学都绝对知道这一款框架，经典中的经典。

Express 是一个 **极简的** Node.js Web 应用框架，非常流行，被广泛用于构建各种 Web 应用和 API。它的主要特点包括：

1. **简洁灵活**：Express 提供了一种轻量级的方式来创建 Web 应用。它不强加太多额外的规范和结构，给开发者提供了很大的灵活性。
2. **中间件架构**：Express 使用中间件的概念来处理 HTTP 请求。中间件是一些函数，它们可以访问请求对象（req）、响应对象（res），以及 Web 应用中处于请求-响应循环流程中的中间件，一般用于完成特定的任务，例如解析请求体、日志记录、身份验证等。
3. **路由系统**：Express 有一个非常强大的路由系统，允许你以非常简洁的方式定义路由，并将不同的 HTTP 请求（GET、POST、PUT、DELETE 等）映射到对应的处理器函数。
4. **社区支持**：由于它的流行，Express 拥有一个庞大的社区，提供了大量的中间件和插件，可以轻松集成进 Express 应用中，极大地增强了其功能。

但究其根源，Express 本身是一个开箱即用，致力于 **快速启动一个服务的框架** ，是比较难以去组织、维护一个相对大型、重量级的企业级项目的。因此目前市面上的很多大型 Node.js 框架，如 Koa.js、Egg.js 都是基于 Express 来封装。

### 使用示例（TS）

下面给一个简单的使用 Express+TS 来启动简单后台的示例。

首先新建一个存放 Express 项目的空目录，用 IDE 打开后在里面安装相关的依赖：

```bash
npm install express
npm install @types/express @types/node typescript ts-node --save-dev
```

然后在根目录运行 `tsc --init` 来生成 `tsconfig.json` 来配置 TS 的编译器，并且可以按需调整其中的配置。

然后创建一个名为 `server.ts` 的文件，并写入以下代码：

```typescript
import express from "express";

const app = express();
const port = 3000;

app.get("/", (req, res) => {
  res.send({ hello: "world" });
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```

设置完毕后，可以使用 TypeScript Node (`ts-node`) 来运行你的服务器：

```bash
ts-node server.ts
```

上面的示例首先导入了 Express，然后创建了一个 Express 应用实例，并为根 URL (`'/'`) 定义了一个 GET 路由，该路由简单地发送一个 JSON 对象作为响应。最后监听在端口 3000 上。此时通过访问 `http://localhost:3000` 可以看到响应。

## Fastify

### 简介

[fastify 官网](https://fastify.dev/)

![fastify主页](https://raw.githubusercontent.com/nonhana/PicGo-Pictures-Store/master/images/image-20231201172120510.png)

Fastify 是一个更现代的、高性能的 Node.js 框架，以速度和低开销为其主要特点。Fastify 的主要特性包括：

1. **高性能**：Fastify 被设计为一个高性能的框架，能够处理大量的请求，同时保持较低的响应时间和资源消耗。
2. **可扩展**：Fastify 通过其提供的钩子（hook）、插件和装饰器（decorator）提供完整的可扩展性。
3. **基于 Schema**：即使这不是强制性的，我们仍建议使用 JSON Schema 来做路由（route）验证及输出内容的序列化，Fastify 在内部将 schema 编译为高效的函数并执行。
4. **日志**：日志是非常重要且代价高昂的。我们选择了最好的日志记录程序来尽量消除这一成本，这就是 Pino!
5. **支持 TypeScript**：开发组努力维护一个 TypeScript 类型声明文件，以便支持不断成长的 TypeScript 社区。
6. **模式驱动**：Fastify 使用 JSON Schema 来处理路由的输入和输出验证，这不仅提供了数据验证，还优化了序列化和解析操作，提高了性能。
7. **丰富的插件生态系统**：与 Express 类似，Fastify 也有一个健康的插件生态系统，可以通过插件来扩展其核心功能。
8. **开发者友好**：Fastify 提供了详细的文档和开发工具，使得开发和维护变得更加容易。

### 使用示例（TS）

和上面的 Express 启动应用类似，我们采用 Fastify 来编写一个差不多的后端服务示例。

首先新建一个存放 Fastify 项目的空目录，用 IDE 打开后在里面安装相关的依赖：

```bash
npm install fastify
npm install @types/node typescript ts-node --save-dev
```

接下来，创建一个名为 `server.ts` 的文件，并写入以下代码：

```typescript
import fastify from "fastify";

const server = fastify();

server.get("/", async (request, reply) => {
  return { hello: "world" };
});

server.listen(3000, (err, address) => {
  if (err) {
    server.log.error(err);
    process.exit(1);
  }
  console.log(`Server listening at ${address}`);
});
```

然后直接在控制台运行 `ts-node server.ts` 就可以启动了。最终的效果和上述 Express 启动的服务类似。
