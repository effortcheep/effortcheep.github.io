---
title: 学习-Effect
date: 2026-04-09 16:33:41
tags: Effect-ts
---


## 效应类型 - Effect Type

`Effect`类型描述的是一个延迟执行的工作流或操作。这意味着，创建`Effect`后，它不会立即运行，而是定义一个程序，该程序可能成功、失败，或者需要一些额外的上下文才能完成。

以下是`Effect`的一般形式

```
         ┌─── Represents the success type
         │        ┌─── Represents the error type
         │        │      ┌─── Represents required dependencies
         ▼        ▼      ▼
Effect<Success, Error, Requirements>
```

这种类型表示存在某种效果：
  + 成功并返回 Success 类型的值。
  + 失败并出现错误类型为 Error
  + 可能需要某些上下文依赖 Requirements 才能执行

**不可变性**: `Effect`值是不可变的，`Effect`中的每个函数都会产生一个新的`Effect`值。
**交互建模**: 这些值本身并不执行任何操作，它们只是对有效的交互进行建模或描述。
**执行**: `Effect`可由 [Effect Runtime System](https://effect.website/docs/runtime/) 执行，该系统会将`Effect`解释为与外部世界的实际交互。理想情况下，此执行发生在应用程序的单个入口点，例如启动`effectful`操作的主函数

>Effect 只负责定义业务逻辑

## 服务管理 - Managing Services

```
                         ┌─── Represents required dependencies
                         ▼
Effect<Success, Error, Requirements>
```
**依赖项声明**：可以直接在函数的类型中指定函数需要哪些服务，从而将依赖项管理的复杂性推入类型系统。
**服务提供**：`Effect.provideService` 用于将服务实现提供给需要它的函数。通过在启动时提供服务，可以确保应用程序的所有部分都能一致地访问所需的服务，从而保持清晰且解耦的架构。


### 如何在 Effect 中管理服务：

1. **创建服务**：定义一个具有独特功能和接口的服务。
2. **使用服务**：在应用程序的功能中访问和使用该服务。
3. **提供服务实施**：提供服务的实际实施，以满足所声明的要求。


### Creating a Service  创建服务
要创建一个新的服务，需要两样东西:
1. **唯一标识符**：字符串 "Logger"
2. **描述操作的类型**：{ log: (msg: string) => Effect.Effect<void> }
```ts
class Logger extends Context.Tag("Logger")<
  Logger,
  { readonly log: (msg: string) => Effect.Effect<void> }
>() {}
```

### 提供服务
使用`Effect.provideService`提供服务
```ts
import { Effect, Context } from "effect"

// Declaring a tag for a service that generates random numbers
class Random extends Context.Tag("MyRandomService")<
  Random,
  { readonly next: Effect.Effect<number> }
>() {}

// Using the service
const program = Effect.gen(function* () {
  const random = yield* Random
  const randomNumber = yield* random.next
  console.log(`random number: ${randomNumber}`)
})

// Providing the implementation
//
//      ┌─── Effect<void, never, never>
//      ▼
const runnable = Effect.provideService(program, Random, {
  next: Effect.sync(() => Math.random())
})

// Run successfully
Effect.runPromise(runnable)
/*
Example Output:
random number: 0.8241872233134417
*/
```


## Default Services  默认服务

Effect 内置了五项预置服务：

```ts
type DefaultServices = Clock | ConfigProvider | Console | Random | Tracer
```
即使程序同时使用了`Clock`和 `Console`，代表效果执行所需服务的`Requirements`参数仍然设置为`never`会自动为我们无缝处理这些服务。

```ts
import { Effect, Clock, Console } from "effect"

//      ┌─── Effect<void, never, never>
//      ▼
const program = Effect.gen(function* () {
  const now = yield* Clock.currentTimeMillis
  yield* Console.log(`Application started at ${new Date(now)}`)
})

Effect.runFork(program)
// Output: Application started at <current time>
```


## Managing Layers 管理服务之间的依赖 抽象出一个Layer的概念

### Creating Layers  创建图层
`Layer`类型的结构如下：
```
        ┌─── The service to be created
        │                ┌─── The possible error
        │                │      ┌─── The required dependencies
        ▼                ▼      ▼
Layer<RequirementsOut, Error, RequirementsIn>
```

`Layer`代表构建`RequirementsOut`（服务）的蓝图。它需要一个`RequirementsIn`（依赖项）作为输入，并且在构建过程中可能会导致`Error`类型的错误。

|Parameter  |	Description |
|----------|------------------------------|
|RequirementsOut  | 要创建的服务或资源|
|Error | 服务构建过程中可能出现的错误类型|
|RequirementsIn	| 构建该服务所需的依赖项|

>为特定服务命名`Layer`时，通常的做法是为“生产”实现添加"Live"后缀，为“测试”实现添加"Test"后缀。例如，对于 Database 服务， DatabaseLive 是您在应用程序中提供的层，而 DatabaseTest 是您在测试中提供的层。


```ts
import { Effect, Context, Layer } from "effect"

// Declaring a tag for the Config service
class Config extends Context.Tag("Config")<
  Config,
  {
    readonly getConfig: Effect.Effect<{
      readonly logLevel: string
      readonly connection: string
    }>
  }
>() {}

// Layer<Config, never, never>
const ConfigLive = Layer.succeed(Config, {
  getConfig: Effect.succeed({
    logLevel: "INFO",
    connection: "mysql://username:password@hostname:port/database_name"
  })
})

// Declaring a tag for the Logger service
class Logger extends Context.Tag("Logger")<
  Logger,
  { readonly log: (message: string) => Effect.Effect<void> }
>() {}

// Layer<Logger, never, Config>
const LoggerLive = Layer.effect(
  Logger,
  Effect.gen(function* () {
    const config = yield* Config
    return {
      log: (message) =>
        Effect.gen(function* () {
          const { logLevel } = yield* config.getConfig
          console.log(`[${logLevel}] ${message}`)
        })
    }
  })
)

// Declaring a tag for the Database service
class Database extends Context.Tag("Database")<
  Database,
  { readonly query: (sql: string) => Effect.Effect<unknown> }
>() {}

// Layer<Database, never, Config | Logger>
const DatabaseLive = Layer.effect(
  Database,
  Effect.gen(function* () {
    const config = yield* Config
    const logger = yield* Logger
    return {
      query: (sql: string) =>
        Effect.gen(function* () {
          yield* logger.log(`Executing query: ${sql}`)
          const { connection } = yield* config.getConfig
          return { result: `Results from ${connection}` }
        })
    }
  })
)
```